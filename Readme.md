# 2048 Game Deployment on AWS EKS using Fargate & ALB Ingress

This project demonstrates deploying the **2048 game application** on **Amazon EKS** using **AWS Fargate** and **AWS Load Balancer Controller (ALB Ingress)**.  
The goal of this project is to understand **real-world Kubernetes ingress, IAM, and cost-aware cloud deployments**.

> ⚠️ Note: The EKS cluster and AWS resources were deleted after deployment to avoid unnecessary AWS charges. Screenshots are included as proof of successful deployment.

---

## Architecture Overview

```

User
|
| HTTP
v
AWS Application Load Balancer (ALB)
|
v
Kubernetes Ingress
|
v
Kubernetes Service
|
v
Pods running on AWS Fargate

````

---

## Technologies Used

- Amazon EKS
- AWS Fargate
- AWS Load Balancer Controller
- Kubernetes (Deployment, Service, Ingress)
- IAM Roles for Service Accounts (IRSA)
- eksctl
- kubectl
- Helm

---

## Prerequisites

- AWS Account
- eksctl installed
- kubectl installed
- Helm installed
- IAM permissions to create EKS, ALB, and IAM roles

---

## Step-by-Step Deployment

### Step 1: Create EKS Cluster
An EKS cluster was created using `eksctl` in the **us-east-1** region.

```bash
eksctl create cluster --name cluster-1 --region us-east-1
````

---

### Step 2: Create Fargate Profile

A Fargate profile was created so pods in the `game-2048` namespace run on **AWS Fargate**.

```bash
eksctl create fargateprofile \
  --cluster cluster-1 \
  --region us-east-1 \
  --name alb-sample-app \
  --namespace game-2048
```

---

### Step 3: Setup AWS Load Balancer Controller (ALB)

#### Download IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

#### Create IAM Policy

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

#### Create IAM Role using IRSA

```bash
eksctl create iamserviceaccount \
  --cluster cluster-1 \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

#### Install ALB Controller using Helm

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=cluster-1 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1
```

---

### Step 4: Deploy 2048 Application

The official Kubernetes YAML was used for deployment.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

This YAML includes:

* Namespace
* Deployment
* Service
* Ingress configuration

---

### Step 5: Verify Deployment

```bash
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
kubectl get deployment -n kube-system aws-load-balancer-controller
```

After successful deployment, an **AWS Application Load Balancer** was automatically created and exposed the application.

---

## Screenshots

> Screenshots are stored in the `screenshots/` folder.

### EKS Cluster

![EKS Cluster](screenshots/eks-cluster.png)

### Fargate Profile

![Fargate Profile](screenshots/fargate-profile.png)

### ALB Created by Ingress

![ALB](screenshots/alb.png)

### Pods Running on Fargate

![Pods](screenshots/pods.png)

### Application Running

![2048 App](screenshots/app-running.png)

---

## Cleanup (Cost Saving)

To avoid AWS charges, all resources were deleted after verification.

```bash
eksctl delete cluster --name cluster-1 --region us-east-1
```

This cleanup removed:

* EKS cluster
* Fargate profiles
* ALB and target groups
* IAM roles and service accounts

---

## Key Learnings

* How Kubernetes Ingress works with AWS ALB
* Running pods on AWS Fargate (serverless Kubernetes)
* Using IAM Roles for Service Accounts (IRSA)
* Real-world cloud cost management
* End-to-end EKS application deployment


