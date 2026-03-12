Helm Multi-Environment Deployment (Step-by-Step Practical)
This project shows how to deploy the same application in multiple environments (Dev, Stage, Prod) using Helm.

Instead of writing different Kubernetes YAML files for each environment, we use:

One Helm Chart
Multiple values files
1️⃣ Prerequisites
Make sure the following tools are installed.

Check Kubernetes
kubectl version --client
AWS EKS Setup
Setup kubectl
a. Download kubectl version 1.20
b. Grant execution permissions to kubectl executable
c. Move kubectl onto /usr/local/bin
d. Test that your kubectl installation was successful

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin 
kubectl version --short --client
Setup eksctl
a. Download and extract the latest release
b. Move the extracted binary to /usr/local/bin
c. Test that your eksclt installation was successful

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
Create an IAM Role and attache it to EC2 instance
Note: create IAM user with programmatic access if your bootstrap system is outside of AWS
IAM user should have access to
IAM
EC2
VPC
CloudFormation

Create your cluster and nodes

eksctl create cluster --name cluster-name  \
--region region-name \
--node-type instance-type \
--nodes-min 2 \
--nodes-max 2 \ 
--zones <AZ-1>,<AZ-2>

example:
eksctl create cluster --name mahima \
   --region us-east-1 \
--node-type t2.small \
Validate your cluster using by creating by checking nodes and by creating a pod

kubectl get nodes
###-----helm install-----

https://github.com/helm/helm/releases

wget https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz

tar -zxvf helm-v3.14.0-linux-amd64.tar.gz

mv linux-amd64/helm /usr/local/bin/helm

chmod 777 /usr/local/bin/helm  # give permissions 
Check Helm
helm version
Check Cluster Connection
kubectl get nodes
If nodes appear, your cluster is ready.

2️⃣ Create Project Folder
Create a new project directory.

mkdir helm-multi-env
cd helm-multi-env
3️⃣ Create Helm Chart
Run the Helm command:

helm create helm-chart
This creates the following structure:

helm-chart/
 ├── Chart.yaml
 ├── values.yaml
 └── templates/
      ├── deployment.yaml
      └── service.yaml
4️⃣ Modify Deployment Template
Open:

helm-chart/templates/deployment.yaml
Find the replicas line and make sure it looks like this:

replicas: {{ .Values.replicaCount }}
Find the container image section:

image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
This tells Helm to take values from values files.

5️⃣ Edit Default values.yaml
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

6️⃣ Create Environment Values Files
Go to project root and create three files.

touch values-dev.yaml
touch values-stage.yaml
touch values-prod.yaml
Project structure should now look like this:

repo/
 ├── helm-chart/
 │    ├── Chart.yaml
 │    ├── values.yaml
 │    └── templates/
 │         ├── deployment.yaml
 │         └── service.yaml
 │
 ├── values-dev.yaml
 ├── values-stage.yaml
 └── values-prod.yaml
7️⃣ Configure Dev Environment
Edit:

values-dev.yaml
Add:

replicaCount: 1

image:
  repository: nginx
  tag: latest
Dev environment uses 1 pod.

8️⃣ Configure Stage Environment
Edit:

values-stage.yaml
Add:

replicaCount: 2

image:
  repository: nginx
  tag: latest
Stage environment uses 2 pods.

9️⃣ Configure Production Environment
Edit:

values-prod.yaml
Add:

replicaCount: 5

image:
  repository: nginx
  tag: latest
Production uses 5 pods.

🔟 Deploy to Dev Environment
Run:

helm install myapp-dev ./helm-chart -f values-dev.yaml
Check pods:

kubectl get pods
You should see 1 pod running.

1️⃣1️⃣ Deploy to Stage Environment
Run:

helm install myapp-stage ./helm-chart -f values-stage.yaml
Check pods:

kubectl get pods
You should see 2 pods running.

1️⃣2️⃣ Deploy to Production Environment
Run:

helm install myapp-prod ./helm-chart -f values-prod.yaml
Check pods:

kubectl get pods
You should see 5 pods running.

1️⃣3️⃣ Verify Services
Check services:

kubectl get svc
1️⃣4️⃣ Check Helm Releases
helm list
Example output:

myapp-dev
myapp-stage
myapp-prod
1️⃣5️⃣ Delete Deployment
If you want to remove deployments:

helm uninstall myapp-dev
helm uninstall myapp-stage
helm uninstall myapp-prod
How This Works
Helm reads templates inside helm-chart/templates.
Helm reads configuration from the selected values file.
Helm replaces template variables.
Final Kubernetes YAML is generated.
Kubernetes creates pods and services.
Final Result
Environment deployments:

myapp-dev    → 1 pod
myapp-stage  → 2 pods
myapp-prod   → 5 pods
Project Workflow
Install required tools (kubectl, Helm, Kubernetes cluster)
Create Helm chart
Configure values.yaml
Create environment values files (dev, stage, prod)
Deploy application using Helm
Kubernetes creates pods based on the selected environment configuration
Same application, different configuration.
