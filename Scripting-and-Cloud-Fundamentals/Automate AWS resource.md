# **Automate AWS resource creation with Bash**

**Goal:** Develop Bash scripts that automate the creation and configuration of essential AWS resources — specifically EC2 instances, Security Groups, and S3 buckets — using the AWS CLI.

---

## **Learning objectives**

- Use the AWS CLI to create and manage cloud resources programmatically

- Write and execute Bash scripts to automate routine infrastructure setup tasks

- Apply best practices for parameterization, error handling, and resource cleanup in automation scripts

- Demonstrate an understanding of security principles by properly configuring IAM credentials and permissions

---

## **Project scenario**

Your team is responsible for provisioning AWS environments for multiple development teams. Manually creating EC2 instances, S3 buckets, and security groups is inefficient and error-prone. Your task is to create a set of Bash scripts that automate this process — from resource creation to validation.

---

## **Project tasks**

### **Task 1: Setup AWS CLI environment**

- Install and configure the AWS CLI

- Verify credentials and region setup using:

```bash
aws sts get-caller-identity
aws configure list
```

### **Task 2: Automate EC2 instance creation**

Write a Bash script named create_ec2.sh that:

- Creates a new EC2 key pair

- Creates a new EC2 instance using a free-tier Amazon Linux 2 AMI

- Tags the instance (e.g., Project=AutomationLab)

- Prints the instance ID and public IP upon creation

Hint: Use aws ec2 create-key-pair and aws ec2 run-instances

### **Task 3: Automate security group creation**

Write a script named create_security_group.sh that:

- Creates a security group (e.g., devops-sg)

- Opens port 22 for SSH access and port 80 for HTTP traffic

- Displays the security group ID and rules

Hint: Use aws ec2 create-security-group and aws ec2 authorize-security-group-ingress

### **Task 4: Automate S3 bucket creation**

Write a script named create_s3_bucket.sh that:

- Creates a uniquely named S3 bucket

- Enables versioning and sets a simple bucket policy

- Uploads a sample text file (e.g., welcome.txt) to the bucket

Hint: Use aws s3api create-bucket and aws s3api put-bucket-versioning

### **Task 5: Cleanup script**

Create a script cleanup_resources.sh that:

- Terminates all resources created by your other scripts

- Uses tagging and filtering (--filters) to identify resources to delete safely

---

## **Expected deliverables**

- create_ec2.sh

- create_security_group.sh

- create_s3_bucket.sh

- cleanup_resources.sh (optional)

- README.md explaining setup and usage

- Screenshots showing successful script execution

---

## **Submission criteria and guidelines**

### **1. Submission format**

Submit a GitHub repository containing:

- All Bash scripts

- Screenshots showing successful execution

- A README.md that includes:

  - The purpose of each script

  - Setup and execution steps

  - Challenges faced and how they were resolved

### **2. Submission process**

- Submit your GitHub repository link via the official Microsoft Forms submission link

- Ensure your repository is public or accessible to reviewers

### **3. Evaluation criteria**

| **Criterion** | **Description** | **Weight** |
| --- | --- | --- |
| Technical accuracy | Scripts execute correctly and perform intended AWS operations | 30% |
| Best practices | Code uses variables, error handling, comments, and clean structure | 25% |
| Documentation quality | README clearly explains setup, purpose, and usage | 25% |
| Completeness | All required scripts and screenshots are included | 20% |

### **4. Submission deadline**

Submissions must be completed by the due date announced by the instructor. Late submissions may incur penalties per course policy.
