## **The "Immutable and Indestructible" pipeline — FinCorp**

**Scenario:** FinCorp requires a highly secure, auditable software supply chain and a disaster recovery plan that ensures their critical database can be restored in a different region within 30 minutes.

---

### **Objectives**

- Implement a secure CI/CD pipeline that produces immutable artifacts

- Demonstrate a cross-region disaster recovery (DR) failover

---

### **Part 1: Artifact pipeline**

#### **1a. Set up AWS CodeArtifact as an upstream proxy**

Configure CodeArtifact to act as a proxy for npm or pip so all dependencies are pulled through a controlled, auditable source.

```bash
# Create a CodeArtifact domain
aws codeartifact create-domain \
  --domain fincorp \
  --encryption-key alias/aws/codeartifact

# Create a repository with an upstream public proxy
aws codeartifact create-repository \
  --domain fincorp \
  --repository fincorp-repo \
  --upstreams repositoryName=npm-store   # or pypi-store for pip

# Get the login token for npm
aws codeartifact login \
  --tool npm \
  --domain fincorp \
  --repository fincorp-repo

# Or for pip
aws codeartifact login \
  --tool pip \
  --domain fincorp \
  --repository fincorp-repo
```

#### **1b. Create the CI/CD pipeline with ECR image scanning and tag immutability**

**Enable tag immutability and image scanning on the ECR repository:**

```bash
# Create ECR repository with tag immutability
aws ecr create-repository \
  --repository-name fincorp-app \
  --image-tag-mutability IMMUTABLE \
  --image-scanning-configuration scanOnPush=true \
  --region us-east-1
```

**GitHub Actions workflow (pipeline.yml):**

```yaml
name: Immutable Artifact Pipeline

on:
  push:
    branches: [main]

env:
  AWS_REGION:     us-east-1
  ECR_REPOSITORY: fincorp-app
  IMAGE_TAG:      ${{ github.sha }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/GitHubActionsRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Configure CodeArtifact for pip/npm
        run: |
          aws codeartifact login \
            --tool pip \
            --domain fincorp \
            --repository fincorp-repo \
            --region ${{ env.AWS_REGION }}

      - name: Build Docker image
        run: |
          docker build -t ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }} .

      - name: Push Docker image to ECR
        run: |
          docker push ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}

      - name: Check ECR image scan results
        run: |
          echo "Waiting for scan results..."
          sleep 30

          CRITICAL=$(aws ecr describe-image-scan-findings \
            --repository-name ${{ env.ECR_REPOSITORY }} \
            --image-id imageTag=${{ env.IMAGE_TAG }} \
            --region ${{ env.AWS_REGION }} \
            --query 'imageScanFindings.findingSeverityCounts.CRITICAL' \
            --output text)

          HIGH=$(aws ecr describe-image-scan-findings \
            --repository-name ${{ env.ECR_REPOSITORY }} \
            --image-id imageTag=${{ env.IMAGE_TAG }} \
            --region ${{ env.AWS_REGION }} \
            --query 'imageScanFindings.findingSeverityCounts.HIGH' \
            --output text)

          echo "Critical vulnerabilities: ${CRITICAL:-0}"
          echo "High vulnerabilities: ${HIGH:-0}"

          if [ "${CRITICAL:-0}" -gt 0 ] || [ "${HIGH:-0}" -gt 0 ]; then
            echo "Build FAILED: High or Critical vulnerabilities found in image."
            exit 1
          fi

          echo "Scan passed — no High or Critical vulnerabilities found."
```

The build fails automatically if any High or Critical vulnerabilities are detected in the image scan. The image tag uses the Git commit SHA, making every artifact immutable and traceable.

#### **1c. Constraint: build fails on High/Critical vulnerabilities**

The pipeline step above queries ECR scan results after push and exits with a non-zero code if vulnerabilities are found. This prevents any vulnerable image from being used in downstream deployment stages.

---

### **Part 2: Disaster recovery**

#### **2a. Deploy RDS database in us-east-1**

```bash
# Create the primary RDS instance
aws rds create-db-instance \
  --db-instance-identifier fincorp-primary \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password <secure-password> \
  --allocated-storage 20 \
  --backup-retention-period 7 \
  --region us-east-1 \
  --tags Key=Project,Value=FinCorp Key=Environment,Value=Production
```

#### **2b. Configure AWS Backup with cross-region copy to us-west-2**

```bash
# Create a backup vault in us-east-1
aws backup create-backup-vault \
  --backup-vault-name fincorp-primary-vault \
  --region us-east-1

# Create a backup vault in us-west-2 (destination)
aws backup create-backup-vault \
  --backup-vault-name fincorp-dr-vault \
  --region us-west-2
```

**Backup plan with cross-region copy (JSON):**

```json
{
  "BackupPlanName": "fincorp-daily-backup",
  "Rules": [
    {
      "RuleName": "DailyBackupWithCrossRegionCopy",
      "TargetBackupVaultName": "fincorp-primary-vault",
      "ScheduleExpression": "cron(0 2 * * ? *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 120,
      "Lifecycle": {
        "DeleteAfterDays": 30
      },
      "CopyActions": [
        {
          "DestinationBackupVaultArn": "arn:aws:backup:us-west-2:<ACCOUNT_ID>:backup-vault:fincorp-dr-vault",
          "Lifecycle": {
            "DeleteAfterDays": 30
          }
        }
      ]
    }
  ]
}
```

```bash
# Create the backup plan
aws backup create-backup-plan \
  --backup-plan file://backup-plan.json \
  --region us-east-1

# Assign RDS resource to the backup plan
aws backup create-backup-selection \
  --backup-plan-id <plan-id> \
  --backup-selection '{
    "SelectionName": "fincorp-rds",
    "IamRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/AWSBackupDefaultServiceRole",
    "Resources": [
      "arn:aws:rds:us-east-1:<ACCOUNT_ID>:db:fincorp-primary"
    ]
  }' \
  --region us-east-1
```

#### **2c. Simulate region failure by deleting the primary database**

```bash
# Simulate region failure — delete the primary RDS instance
aws rds delete-db-instance \
  --db-instance-identifier fincorp-primary \
  --skip-final-snapshot \
  --region us-east-1
```

Document the timestamp of deletion. This simulates a full region failure for the primary database. Take a screenshot confirming the instance is deleted.

#### **2d. Restore the database in us-west-2 from the copied backup**

```bash
# List available recovery points in the DR vault
aws backup list-recovery-points-by-backup-vault \
  --backup-vault-name fincorp-dr-vault \
  --region us-west-2

# Restore the database from the most recent recovery point
aws backup start-restore-job \
  --recovery-point-arn arn:aws:backup:us-west-2:<ACCOUNT_ID>:recovery-point:<recovery-point-id> \
  --iam-role-arn arn:aws:iam::<ACCOUNT_ID>:role/AWSBackupDefaultServiceRole \
  --metadata '{
    "DBInstanceIdentifier": "fincorp-restored",
    "DBInstanceClass": "db.t3.micro",
    "Engine": "postgres",
    "MultiAZ": "false"
  }' \
  --region us-west-2

# Monitor restore job status
aws backup describe-restore-job \
  --restore-job-id <restore-job-id> \
  --region us-west-2
```

Record the total time from deletion to successful restore. The target is under 30 minutes. Take a screenshot of the restored RDS instance showing status as "available" in us-west-2.

---

### **Required deliverables**

```
fincorp-pipeline/
├── .github/
│   └── workflows/
│       └── pipeline.yml
├── Dockerfile
├── app/
│   └── (application source code)
├── infrastructure/
│   ├── ecr.tf                   # or ecr CloudFormation template
│   ├── rds.tf
│   └── backup-plan.json
├── screenshots/
│   ├── ecr_tag_immutability.png
│   ├── ecr_scan_results.png
│   ├── pipeline_scan_failure.png
│   ├── pipeline_success.png
│   ├── rds_primary_deleted.png
│   ├── backup_copy_in_us_west_2.png
│   ├── restore_job_running.png
│   └── rds_restored_us_west_2.png
└── documentation/
    └── fincorp_dr_report.md
```

---

### **DR report structure (fincorp_dr_report.md)**

**Section 1 — Pipeline security:**

- How CodeArtifact acts as a controlled upstream proxy

- ECR tag immutability and what it prevents

- ECR image scanning setup and the vulnerability gate in the pipeline

- Evidence of a build failing due to a High/Critical vulnerability

**Section 2 — Disaster recovery:**

- RDS configuration in us-east-1

- AWS Backup plan and cross-region copy configuration

- Step-by-step simulation of the region failure

- Restore procedure and time taken

- RTO (Recovery Time Objective) achieved vs. the 30-minute target

**Section 3 — Recommendations:**

- How to automate the failover further (e.g., Route 53 health checks and DNS failover)

- How to test DR regularly without disrupting production

---

### **Live walkthrough checklist**

- Show CodeArtifact domain and repository configured as upstream proxy

- Trigger the pipeline and show a successful build with clean scan results

- Deliberately introduce a vulnerable dependency and show the pipeline failing at the scan step

- Show ECR repository with tag immutability enabled

- Show the AWS Backup plan with cross-region copy rule to us-west-2

- Show an existing recovery point in the us-west-2 DR vault

- Delete the primary RDS instance to simulate region failure

- Start the restore job in us-west-2 and monitor until complete

- Show the restored database in us-west-2 with status "available"

- Record and state the total RTO achieved

---

### **Tips**

- Use the Git commit SHA as the image tag to guarantee immutability and full traceability back to the exact code that was built

- Test the vulnerability gate by temporarily adding a known vulnerable package version (e.g., an old version of requests or lodash) then reverting it after demonstrating the failure

- Trigger a manual AWS Backup job immediately after setup so you have a recovery point ready for the DR simulation without waiting for the nightly schedule

- Use Route 53 with health checks in production to automate DNS failover to us-west-2 during a real region outage

- Store the backup plan JSON in version control so the DR configuration is itself auditable and reproducible
