# Helm Multi-Environment Deployment (Step-by-Step Practical)

This project demonstrates how to deploy the same application in multiple environments (**Dev, Stage, Production**) using **Helm on Kubernetes (EKS)**.

Instead of writing different Kubernetes YAML files for each environment, we use:

* One Helm Chart
* Multiple Values Files

---

# 1️⃣ Prerequisites

Make sure the following tools are installed.

## Check Kubernetes

kubectl version --client

---

# Setup kubectl

Download kubectl:

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

Give permissions:

chmod +x ./kubectl

Move kubectl to system path:

mv ./kubectl /usr/local/bin

Verify installation:

kubectl version --short --client

---

# Setup eksctl

Download and install eksctl:

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

Check version:

eksctl version

---

# Create IAM Role and Attach to EC2

Create IAM user with programmatic access if your system is outside AWS.

Required permissions:

* IAM
* EC2
* VPC
* CloudFormation

---

# Create Kubernetes Cluster

eksctl create cluster --name cluster-name 
--region region-name 
--node-type instance-type 
--nodes-min 2 
--nodes-max 2 
--zones <AZ-1>,<AZ-2>

Example:

eksctl create cluster --name mahima 
--region us-east-1 
--node-type t2.small

Verify cluster:

kubectl get nodes

---

# 2️⃣ Install Helm

Download Helm:

wget https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz

Extract:

tar -zxvf helm-v3.14.0-linux-amd64.tar.gz

Move binary:

mv linux-amd64/helm /usr/local/bin/helm

Give permission:

chmod 777 /usr/local/bin/helm

Check version:

helm version

Check cluster connection:

kubectl get nodes

If nodes appear, the cluster is ready.

---

# 3️⃣ Create Project Folder

mkdir helm-multi-env

cd helm-multi-env

---

# 4️⃣ Create Helm Chart

helm create helm-chart

Generated structure:

helm-chart/

├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml

---

# 5️⃣ Modify Deployment Template

Open:

helm-chart/templates/deployment.yaml

Update replicas section:

replicas: {{ .Values.replicaCount }}

Update image section:

image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"

This allows Helm to read values from values files.

---

# 6️⃣ Edit Default values.yaml

Open:

helm-chart/values.yaml

Example configuration:

replicaCount: 2

image:
repository: nginx
tag: latest

service:
type: ClusterIP
port: 80

This file contains default configuration.

---

# 7️⃣ Create Environment Values Files

Create environment specific configuration files.

touch values-dev.yaml

touch values-stage.yaml

touch values-prod.yaml

Project structure:

repo/

├── helm-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│        ├── deployment.yaml
│        └── service.yaml

├── values-dev.yaml
├── values-stage.yaml
└── values-prod.yaml

---

# 8️⃣ Configure Dev Environment

Edit:

values-dev.yaml

replicaCount: 1

image:
repository: nginx
tag: latest

Dev environment runs **1 pod**.

---

# 9️⃣ Configure Stage Environment

Edit:

values-stage.yaml

replicaCount: 2

image:
repository: nginx
tag: latest

Stage environment runs **2 pods**.

---

# 🔟 Configure Production Environment

Edit:

values-prod.yaml

replicaCount: 5

image:
repository: nginx
tag: latest

Production runs **5 pods**.

---

# 1️⃣1️⃣ Deploy to Dev Environment

helm install myapp-dev ./helm-chart -f values-dev.yaml

Check pods:

kubectl get pods

Expected: **1 pod**

---

# 1️⃣2️⃣ Deploy to Stage Environment

helm install myapp-stage ./helm-chart -f values-stage.yaml

Check pods:

kubectl get pods

Expected: **2 pods**

---

# 1️⃣3️⃣ Deploy to Production Environment

helm install myapp-prod ./helm-chart -f values-prod.yaml

Check pods:

kubectl get pods

Expected: **5 pods**

---

# 1️⃣4️⃣ Verify Services

kubectl get svc

---

# 1️⃣5️⃣ Check Helm Releases

helm list

Example output:

myapp-dev
myapp-stage
myapp-prod

---

# 1️⃣6️⃣ Delete Deployment

helm uninstall myapp-dev

helm uninstall myapp-stage

helm uninstall myapp-prod

---

# How Helm Multi-Environment Deployment Works

1. Helm reads templates inside `helm-chart/templates`
2. Helm reads configuration from the selected values file
3. Helm replaces template variables
4. Kubernetes YAML manifests are generated
5. Kubernetes creates pods and services

---

# Final Result

Environment Deployments:

myapp-dev → 1 pod
myapp-stage → 2 pods
myapp-prod → 5 pods

---

# Project Workflow

Install required tools (kubectl, Helm, Kubernetes cluster)

Create Helm chart

Configure values.yaml

Create environment-specific values files (Dev, Stage, Prod)

Deploy application using Helm

Kubernetes creates pods based on environment configuration

---

# Architecture Flow

Code
↓
Docker Build
↓
Docker Hub
↓
Helm Chart
↓
Kubernetes (EKS)
↓
NodePort / LoadBalancer
↓
Browser
