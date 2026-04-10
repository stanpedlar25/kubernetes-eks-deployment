Kubernetes Backend Deployment on AWS EKS
Author: David Koduah  
Series: AWS Compute — Amazon EKS (Parts 1–4)  
Region: eu-west-2 (London)
---
Project Overview
A 4-part project deploying a containerised Flask backend application onto Amazon EKS (Elastic Kubernetes Service). Covers the full end-to-end workflow: provisioning a Kubernetes cluster, containerising application code, storing images in Amazon ECR, writing Kubernetes manifest files, and deploying and verifying running pods.
---
Architecture
```
GitHub Repository (Flask Backend Code)
        │
        ▼
   EC2 Instance (eksctl control node)
        │
        ├── Docker Build → Container Image
        │
        ▼
Amazon ECR (Elastic Container Registry)
        │
        ▼
Amazon EKS Cluster (eu-west-2)
        │
        ├── Node Group (2x t3.medium EC2 nodes)
        │
        ├── Deployment Manifest → 3 Replica Pods
        │
        └── Service Manifest (NodePort :8080) → Traffic Routing
```
---
What Was Built
Part 1 — Launch a Kubernetes Cluster
Provisioned an EC2 instance and installed `eksctl`
Configured IAM role with AdministratorAccess for EKS cluster creation
Created EKS cluster in `eu-west-2` using `eksctl create cluster`
CloudFormation automatically provisioned: VPC, subnets, security groups, NAT gateway, route tables
Two CloudFormation stacks created — one for the cluster, one for the node group (separated for independent troubleshooting)
Verified 2 nodes running in the EKS console with status Ready
Tested self-healing: manually deleted node instances — EKS automatically recreated them within minutes
Part 2 — Set Up Kubernetes Deployment
Installed Git on EC2 instance and cloned Flask backend repository
Installed Docker and resolved EC2-USER permissions error by adding user to docker group
Built container image: `docker build -t nextwork-flask-backend .`
Authenticated with Amazon ECR and pushed image to private repository
Verified image available in ECR console (tagged: `latest`)
Part 3 — Create Kubernetes Manifests
Wrote Deployment manifest (`flask-deployment.yaml`): 3 replicas, pulls image from ECR
Wrote Service manifest (`flask-service.yaml`): NodePort type, exposes port 8080
Annotated deployment manifest components for documentation and understanding
Verified manifest structure in nano editor within EC2 terminal
Part 4 — Deploy Backend & Verify
Installed `kubectl` on EC2 instance
Applied both manifests: `kubectl apply -f flask-deployment.yaml` and `kubectl apply -f flask-service.yaml`
Configured IAM access to view EKS cluster nodes in console
Verified pods running in each node via EKS console
Confirmed full pod lifecycle events: Pulling → Pulled → Created → Started
---
Key Files
Deployment Manifest (`flask-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextwork-flask-backend
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nextwork-flask-backend
  template:
    metadata:
      labels:
        app: nextwork-flask-backend
    spec:
      containers:
        - name: nextwork-flask-backend
          image: <account-id>.dkr.ecr.eu-west-2.amazonaws.com/nextwork-flask-backend:latest
          ports:
            - containerPort: 8080
```
Service Manifest (`flask-service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nextwork-flask-backend
spec:
  selector:
    app: nextwork-flask-backend
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
```
---
Technologies Used
Tool / Service	Purpose
Amazon EKS	Managed Kubernetes cluster
Amazon ECR	Container image registry
Amazon EC2	Control node for cluster management
eksctl	CLI tool for EKS cluster creation
kubectl	CLI tool for applying manifests and managing pods
Docker	Build and tag container images
CloudFormation	Auto-provisioned networking stack for EKS
IAM	Roles and permissions for EKS and ECR access
Git	Clone backend application code
---
Key Concepts Demonstrated
Pods — smallest deployable unit in Kubernetes; containers run inside pods
Deployments — manage desired state, replica count, and rolling updates
Services — expose applications and route external traffic to pods
Node Groups — EC2 instances managed by EKS to run container workloads
Replicas — 3 identical pod copies running across nodes for high availability
ECR Integration — EKS pulls images directly from ECR with minimal authentication setup
Self-healing — deleted nodes are automatically recreated by the node group Auto Scaling group
---
Troubleshooting Encountered
Error	Cause	Resolution
`eksctl: command not found`	eksctl not installed	Downloaded eksctl binary and added to PATH
Docker permission denied	EC2-USER not in docker group	`sudo usermod -aG docker ec2-user` then reconnected
EKS console access denied	IAM access entry not configured	Added IAM access entry for cluster via EKS console
---
Related Projects
AWS CI/CD Multi-Environment Pipeline
WordPress 3-Tier Architecture on AWS
EC2 Instance Scheduler — Lambda & EventBridge
AWS VPC Networking Project — Terraform
