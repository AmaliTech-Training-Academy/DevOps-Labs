# **IaC with Terraform**

**Objective:** Use Terraform to define and deploy foundational AWS infrastructure and configure a remote backend in Amazon S3 with DynamoDB state locking.

---

## **Instructions**

- Install Terraform and configure AWS credentials

- Create main.tf, variables.tf, and outputs.tf

- Define the following resources:

  - VPC

  - Public subnet

  - Internet Gateway and route

  - Security group (SSH port 22 from your IP, HTTP port 80 from 0.0.0.0/0)

  - EC2 instance (t2.micro)

- Configure a remote backend: S3 bucket for state storage and DynamoDB lock table (mandatory)

- Run terraform init, terraform plan, terraform apply — verify resources — then terraform destroy

- Use AWS Free Tier or sandbox environment

---

## **Deliverables**

- All .tf files

- Screenshots of terraform plan, terraform apply, and terraform destroy

- AWS Console screenshot showing the EC2 instance

- Proof of remote backend configuration (S3 bucket and DynamoDB lock table)

---

## **Submission requirements**

| **Item** | **Format** | **Contents** |
| --- | --- | --- |
| GitHub repository | .tf files and screenshots | All Terraform configuration files |
| Tools evidence | Screenshot or version output | Terraform v1.5+ |
| Resources | Console screenshots | VPC, subnet, IGW, security group, EC2 |
| Backend proof | Screenshot or config snippet | S3 backend and DynamoDB lock table |
| Lifecycle evidence | Screenshots | Successful create and destroy |

---

