# AWS + Jenkins + EKS Deployment

## Project Overview

This project demonstrates a complete CI/CD workflow using:

- AWS EKS (Elastic Kubernetes Service)
- Terraform (Infrastructure as Code)
- Jenkins Pipeline (CI/CD Automation)
- Docker + DockerHub
- Kubernetes Deployments and Services

The goal is to provision infrastructure using Terraform, build and push Docker images using Jenkins, and deploy applications automatically into the EKS cluster.

---

## Architecture

```text
GitHub Repository
        ↓
Jenkins Pipeline
        ↓
Terraform → AWS Infrastructure (VPC + EKS)
        ↓
Docker Build → DockerHub Push
        ↓
kubectl Deploy → EKS Cluster
        ↓
Application exposed via Service (LoadBalancer / NodePort)
```

---

## Infrastructure Components

### AWS Resources

- VPC
- Public Subnets
- Internet Gateway
- Route Tables
- EKS Cluster
- EKS Managed Node Group
- Security Groups
- IAM Roles
- S3 Backend for Terraform State
- DynamoDB Table for State Locking

---

## Folder Structure

```text
AWS-JENKINS-EKS-Deployment/
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── terraform.tfvars
│
├── K8S/
│   ├── deployment.yaml
│   └── service.yaml
│
├── Dockerfile
├── Jenkinsfile
└── README.md
```

---

## Jenkins Pipeline Stages

### 1. Git Clone

Pull latest source code from GitHub.

### 2. Infrastructure Apply

Run Terraform commands:

```bash
terraform init
terraform validate
terraform plan
terraform apply -auto-approve
```

### 3. Docker Build + Push

- Build Docker image
- Login to DockerHub
- Push latest image

### 4. EKS Deployment

```bash
aws eks update-kubeconfig --name my-cluster --region ap-south-1
kubectl apply -f K8S/deployment.yaml
kubectl apply -f K8S/service.yaml
```

---

## Prerequisites

Before running the project:

### Jenkins Server

Install:

- Java
- Jenkins
- Docker
- Terraform
- AWS CLI
- kubectl
- Git

### Jenkins Credentials

Add:

- AWS Credentials → `AWS_Cred`
- DockerHub Credentials → `Dockerhub-creds`

---

## Terraform Backend Setup

### Create S3 Bucket

```bash
aws s3 mb s3://your-terraform-state-bucket --region ap-south-1
```

### Create DynamoDB Lock Table

```bash
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```

---

## Kubernetes Deployment Example

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foodweb1-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: foodweb1
  template:
    metadata:
      labels:
        app: foodweb1
    spec:
      containers:
      - name: foodweb1
        image: ramesh040183/foodweb1:latest
        ports:
        - containerPort: 80
```

---

## Service Example

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: foodweb1-service
spec:
  type: LoadBalancer
  selector:
    app: foodweb1
  ports:
    - port: 80
      targetPort: 80
```

---

## Verification Commands

### Check Nodes

```bash
kubectl get nodes
```

### Check Deployments

```bash
kubectl get deployments
```

### Check Services

```bash
kubectl get svc
```

### Check Pods

```bash
kubectl get pods
```

---

## Access Application

### If Service Type = LoadBalancer

Open:

```text
http://EXTERNAL-IP
```

### If Service Type = NodePort

Open:

```text
http://EC2-PUBLIC-IP:NODEPORT
```

---

## Author

Project by Rameshwar

AWS + EKS + Jenkins CI/CD Project

