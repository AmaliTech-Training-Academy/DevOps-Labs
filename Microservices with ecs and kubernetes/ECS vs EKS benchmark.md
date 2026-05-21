## **ShopNow: ECS vs EKS benchmark project**

**Role:** DevOps Engineer | **Company:** ShopNow (e-commerce startup)

---

### **Project overview**

The CTO wants to benchmark AWS ECS (Fargate) against Amazon EKS to decide which orchestrator best suits the company. You will containerize a multi-tier application and deploy it to both platforms, then compare the results.

**Application stack:**

- Frontend (Python/Node.js)

- Backend API (Python/Node.js)

- Redis (caching)

- PostgreSQL (database)

---

### **Objectives**

- Containerize a multi-tier application

- Deploy the same application to both Amazon ECS (Fargate) and Amazon EKS

- Implement service discovery and load balancing in both environments

---

### **Instructions**

#### **1. Containerization**

**Tasks:**

- Create Dockerfiles for the frontend and backend

- Create a docker-compose.yml to verify the full stack runs locally before cloud deployment

**Required files:**

```
app/
├── frontend/
│   ├── app.py          # or app.js
│   ├── requirements.txt  # or package.json
│   └── Dockerfile
├── backend/
│   ├── app.py          # or app.js
│   ├── requirements.txt  # or package.json
│   └── Dockerfile
└── docker-compose.yml
```

**Sample docker-compose.yml:**

```yaml
services:

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - BACKEND_URL=http://backend:5000
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://admin:password@postgres:5432/shopnow
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: shopnow
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

#### **2. Infrastructure as Code (IaC)**

Use Terraform or CloudFormation to provision:

- VPC with public and private subnets across two Availability Zones

- ECS Cluster (Fargate)

- EKS Cluster

**Required Terraform file structure:**

```
infrastructure/
├── main.tf
├── variables.tf
├── outputs.tf
├── vpc.tf
├── ecs.tf
└── eks.tf
```

**Sample VPC Terraform block:**

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "shopnow-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    Project     = "ShopNow"
    Environment = "benchmark"
  }
}
```

#### **3. ECS deployment**

**Tasks:**

- Create ECS Task Definitions for frontend, backend, Redis, and PostgreSQL

- Create ECS Services for each component

- Ensure the frontend communicates with the backend via:

  - AWS Cloud Map (service discovery), or

  - An internal Application Load Balancer

**Sample ECS Task Definition (backend):**

```json
{
  "family": "shopnow-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "backend",
      "image": "<your-ecr-repo>/backend:latest",
      "portMappings": [
        { "containerPort": 5000, "protocol": "tcp" }
      ],
      "environment": [
        { "name": "DATABASE_URL", "value": "postgresql://admin:password@<rds-endpoint>:5432/shopnow" },
        { "name": "REDIS_URL",    "value": "redis://<elasticache-endpoint>:6379" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group":  "/ecs/shopnow-backend",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

**ECS service discovery with Cloud Map:**

- Create a private DNS namespace (e.g., shopnow.local)

- Register each service so backend.shopnow.local resolves internally

- Frontend connects to backend using the DNS name

#### **4. EKS deployment**

**Tasks:**

- Create Kubernetes manifests for all four components

- Deploy to EKS using kubectl apply

- Use a Kubernetes Ingress (AWS Load Balancer Controller) for external traffic

**Required Kubernetes manifest files:**

```
k8s/
├── frontend-deployment.yml
├── frontend-service.yml
├── backend-deployment.yml
├── backend-service.yml
├── redis-deployment.yml
├── redis-service.yml
├── postgres-deployment.yml
├── postgres-service.yml
└── ingress.yml
```

**Sample backend deployment manifest:**

```yaml
# backend-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: <your-ecr-repo>/backend:latest
          ports:
            - containerPort: 5000
          env:
            - name: DATABASE_URL
              value: "postgresql://admin:password@postgres:5432/shopnow"
            - name: REDIS_URL
              value: "redis://redis:6379"
```

**Sample backend service manifest:**

```yaml
# backend-service.yml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
    - port: 5000
      targetPort: 5000
  type: ClusterIP
```

**Sample Ingress manifest:**

```yaml
# ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shopnow-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 3000
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 5000
```

#### **5. Resiliency testing**

Manually kill a container or pod in both environments and demonstrate automatic recovery.

**ECS resiliency test:**

```bash
# Stop a running ECS task manually
aws ecs stop-task \
  --cluster shopnow-cluster \
  --task <task-id> \
  --reason "Resiliency test"

# Watch ECS automatically launch a replacement task
aws ecs describe-services \
  --cluster shopnow-cluster \
  --services shopnow-backend
```

**EKS resiliency test:**

```bash
# Delete a running pod manually
kubectl delete pod <backend-pod-name>

# Watch Kubernetes automatically recreate the pod
kubectl get pods -w
```

Document the time taken for each platform to recover and include screenshots showing the replacement container or pod becoming healthy.

---

### **Required deliverables**

```
shopnow-benchmark/
├── app/
│   ├── frontend/
│   ├── backend/
│   └── docker-compose.yml
├── infrastructure/
│   ├── main.tf
│   ├── vpc.tf
│   ├── ecs.tf
│   └── eks.tf
├── k8s/
│   ├── frontend-deployment.yml
│   ├── backend-deployment.yml
│   ├── redis-deployment.yml
│   ├── postgres-deployment.yml
│   └── ingress.yml
├── ecs/
│   └── task-definitions/
├── screenshots/
│   ├── local_docker_compose.png
│   ├── ecs_services_running.png
│   ├── eks_pods_running.png
│   ├── ecs_resiliency_recovery.png
│   ├── eks_resiliency_recovery.png
│   └── load_balancer_endpoints.png
└── documentation/
    └── benchmark_report.md
```

---

### **Benchmark comparison document**

Include the following comparison in your benchmark_report.md:

| **Criteria** | **ECS (Fargate)** | **EKS** |
| --- | --- | --- |
| Setup complexity | Low — managed control plane | High — more configuration required |
| Deployment method | Task definitions and services | Kubernetes manifests |
| Service discovery | Cloud Map or internal ALB | Kubernetes DNS (ClusterIP services) |
| Load balancing | ALB via listener rules | AWS Load Balancer Controller and Ingress |
| Auto-recovery time | Time observed during test | Time observed during test |
| Scaling mechanism | ECS Service Auto Scaling | Horizontal Pod Autoscaler |
| Cost model | Pay per task vCPU and memory | Pay per EC2 node or Fargate pod |
| Operational overhead | Low | Higher — requires Kubernetes expertise |
| Best suited for | Simpler workloads, smaller teams | Complex microservices, large teams |

---

### **Submission requirements**

- GitHub repository containing all code, manifests, and IaC files

- Screenshots of every major step (local run, ECS services, EKS pods, load balancer endpoints, resiliency recovery)

- Completed benchmark comparison document

- Live walkthrough demonstrating:

  - Local Docker Compose run

  - ECS services running and accessible

  - EKS pods running and accessible

  - Resiliency recovery on both platforms

  - Service discovery working in both environments

---

### **Tips**

- Push all images to Amazon ECR before deploying to ECS or EKS

- Use IAM roles for ECS task execution and EKS node groups — never hardcode credentials

- Install the AWS Load Balancer Controller on EKS before applying the Ingress manifest

- Use kubectl get events to diagnose pod startup issues on EKS

- Tag all resources with Project=ShopNow for easy identification and cost tracking during cleanup

- Run terraform destroy after the live walkthrough to avoid ongoing charges
