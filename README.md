# **Deployment of Containerized Web Application (vprofile) on AWS EKS Cluster AWS with CLB, DNS & TLS (Terraform Automated)**

## 📌 Overview

This project provisions a production-ready private Amazon EKS cluster using Infrastructure as Code (Terraform) and deploys a containerized Web Application using Kubernetes with:

- AWS Classic Balancer Controller (CLB)

- EKS Nginx Ingress 

- TLS termination using ACM

- DNS (Domain Name)

- Bastion host for secure access

The entire infrastructure layer is automated using Terraform.

## Architecture
Core AWS Services Used
- Amazon Web Services
- Amazon EKS
- AWS Classic Load Balancer
- AWS IAM
- AWS Certificate Manager

<img src="./images/vpc.png">
<img src="./images/eks.png">

## Infrastructure Provisioned with Terraform
### Networking (Custom VPC)
- VPC
- Public Subnets
- Private Subnets
- Internet Gateway (IGW)
- NAT Gateway
- Route Tables
- Security Groups

#### Architecture model:
```
Public Subnet:
  - Bastion Host
  - NAT Gateway
  - CLB

Private Subnet:
  - EKS Worker Nodes
```

### Private EKS Cluster
- Private Endpoint Enabled
- IAM Roles & Policies
- OIDC Provider
- IRSA (IAM Roles for Service Accounts)
- Managed Node Groups

#### Security-first design:
- No public access to worker nodes
- Access only via Bastion host

### Bastion Host
Used for:
- Secure SSH access
- Kubectl access to private cluster
- Helm installation
- Controller installation

# **Prerequisites Setup for This Repository (AWS CLI + Terraform via Chocolatey)**

Before running this EKS Terraform project, install the required tools on your system using Chocolatey.

# **1. Install Chocolatey (if not already installed)**

Open PowerShell as Administrator and install Chocolatey:
```shell

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; ` iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

```

Verify installation:

```shell
choco -v
```

# **2. Install AWS CLI using Chocolatey**

Install AWS CLI:
```shell
choco install awscli -y
```
Verify:
```shell
aws --version
```
# **3. Install Terraform using Chocolatey**

Install Terraform:
```shell
choco install terraform -y
```
Verify:
```shell
terraform -version
```
# **EKS Project Run Steps (Terraform)**

## **1.Clone the repository to local: create a empty directory in local .Then clone it**

or 
## **in vs code-->click on terminal-->new terminal-->select git bash-->change to local directory --> run git clone command-->after cloning finished--->cick on file-->open Folder-->select your cloned repository**
```shell
git clone https://github.com/RupaEnggVille/PRODUCTION-EKS.git

cd PRODUCTION-EKS
```
## **2. Go to Terraform working directory**

Based on your instructions:
```shell
ls
cd eks-project
ls
cd terraform
ls
cd eks
```
Make sure this folder contains: main.tf ,eks.tf,vpc.tf,provider.tf,dev.tfvars

## **3. Configure AWS CLI (mandatory):** for that go to browser create a IAM user (eks-user) in aws console for this project save access key and secret keys locally
```shell
aws configure

Set:

AWS Access Key : give your access key
Secret Key  : give your secret key
Region → us-east-1
json
```
## **4. Create S3 backend bucket (if not already created)**

Through aws console or through command
```shell
aws s3 mb s3://your-terraform-state-bucket --region us-east-1
```

in **backend.tf** change the bucket name 

bucket       = "backend-bucket-final-6526"   #create s3 bucket through aws console .and replace bucket name here

Update the region based on your AWS region. in backend block 


### In dev.tfvars 

Change Ami id of Ubuntu Server Based on region

ami_id        = "ami-091138d0f0d41ff90"  #replace with your ubuntu ami id based on region

Instance type based on your project size

instance_type = "t3.medium"  #change instance type

aws_region                = "us-east-1"   #change region

instance_types   = ["t3.medium"]  

if reguired change 

kubernetes_version        = "1.34" also


## **5. Create EC2 Key Pair (VERY IMPORTANT)**

Your project uses: check it in ec2.tf file which block you are using

data "aws_key_pair"

So key MUST already exist in AWS.**(create manually through aws console)**

If you use resource "aws_key_pair"
Generate through ssh-keygen

### Step 1: Create local key
```shell
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ec2_keypair
```
### Step 2: Import into AWS
```shell
aws ec2 import-key-pair \
  --key-name ec2_keypair \
  --public-key-material fileb://~/.ssh/ec2_keypair.pub \
  --region us-east-1
```  
### Step 3: Verify
```shell
aws ec2 describe-key-pairs --key-names ec2_keypair
```  
## **6. Initialize Terraform**
```shell
terraform init
```  
This will: download AWS provider ,initialize backend (S3) ,prepare modules

## **7. Validate configuration**
```shell
terraform validate
```
## **8. Plan infrastructure**
```shell
terraform plan -var-file="dev.tfvars"    #because all global configuration settings are defined in dev.tfvars. If you run only terraform plan, it will prompt you to enter values manually.”
```
We use dev.tfvars with terraform plan so Terraform already knows all values. If we don’t use it, Terraform will stop and ask us to enter them one by one.

## **9. Apply infrastructure**
```shell
terraform apply -var-file="dev.tfvars"

Type:yes
```
After resource creation completes, verify in the AWS Console that the bastion server, cluster, and VPCs ,IAM Roles are available. The process usually takes 10–15 minutes. Then copy the bastion server’s public IP address and launch a new Git Bash session.


## **10. Post Deployment (Bastion Access & Kubernetes Setup)**

Because cluster is private:SSH into Baston server 
```shell
ssh -i Downloads/ec2_keypair.pem ubuntu@bastion_public_ip
```

(example: ssh -i Downloads/ec2_keypair.pem ubuntu@(bastion_public_ip))

### **Configure Kubernetes access**
```shell
aws configure    #enter here access keys and secret keys ,region and json

AWS Access Key : give your saved access key

Secret Key  : give your saved secret key

Region → us-east-1

Output → json
```
```shell
aws eks update-kubeconfig --region us-east-1 --name dev-eks-demo
```
**Verify:**
```shell
kubectl get nodes
```
## **11. Install Helm (inside bastion)**
```shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```




# NGINX Ingress Controller Setup on Amazon EKS

This guide explains how to install an Internet-facing NGINX Ingress Controller on an Amazon EKS cluster using Helm, expose applications using Kubernetes Ingress resources, and clean up the installation when no longer needed.

---

# Prerequisites

Before starting, ensure the following are installed and configured:

- Kubernetes Cluster (Amazon EKS)
- kubectl
- Helm
- eksctl
- AWS CLI
- IAM permissions for EKS and ELB creation

Verify cluster access:

```bash
kubectl get nodes

Install NGINX Ingress Controller
Add Helm Repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
Explanation

Adds the official NGINX Ingress Helm chart repository to Helm.

Update Helm Repositories
helm repo update
Explanation

Fetches the latest chart information from configured Helm repositories.

Install Internet-Facing NGINX Ingress Controller
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-scheme"="internet-facing" \
  --set controller.service.type=LoadBalancer
Explanation

This command installs the NGINX Ingress Controller and creates an AWS Elastic Load Balancer (ELB).

Parameters
Parameter	Description
helm upgrade --install	Installs the chart if not present, otherwise upgrades it
ingress-nginx	Release name
ingress-nginx/ingress-nginx	Helm chart name
--namespace ingress-nginx	Installs into ingress-nginx namespace
--create-namespace	Creates namespace if it does not exist
aws-load-balancer-scheme=internet-facing	Creates a public ELB
controller.service.type=LoadBalancer	Exposes controller externally
Verify Installation
kubectl get svc -n ingress-nginx
Expected Output

You should see:

ingress-nginx-controller   LoadBalancer   <external-ip>

AWS will provision an ELB automatically.

Create Ingress Resource

Example ingress manifest:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vpro-ingress
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: vprofile.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vproapp-service
            port:
              number: 8080
Apply Ingress
kubectl apply -f ingress.yaml
Verify Ingress
kubectl get ingress

Expected output:

NAME           CLASS   HOSTS                  ADDRESS
vpro-ingress   nginx   vprofile.example.com  <elb-hostname>
Configure DNS

Create a DNS CNAME record:

Type	Name	Value
CNAME	vprofile	ELB hostname

Example:

vprofile.example.com -> abc123.us-east-1.elb.amazonaws.com
Test Application

Using curl:

curl http://vprofile.example.com

Or open in browser:

http://vprofile.example.com
Persistent Volume Claim Example

Example PVC manifest:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2
  resources:
    requests:
      storage: 3Gi

Apply PVC:

kubectl apply -f pvc.yaml
Verify PVC
kubectl get pvc

If status remains Pending, ensure:

EBS CSI Driver is installed
A pod is consuming the PVC
StorageClass exists
Install AWS EBS CSI Driver
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster <cluster-name> \
  --force

Verify:

kubectl get pods -n kube-system | grep ebs
Delete NGINX Ingress Controller
Remove Helm Release
helm uninstall ingress-nginx -n ingress-nginx
Delete Namespace
kubectl delete namespace ingress-nginx
Useful Troubleshooting Commands
Check Ingress
kubectl describe ingress
Check Services
kubectl get svc -A
Check Storage Classes
kubectl get sc
Check Persistent Volumes
kubectl get pv
Check PVC Events
kubectl describe pvc <pvc-name>
Notes
NGINX Ingress Controller is different from AWS ALB Ingress Controller.
Use ingressClassName: nginx for NGINX ingress.
Do not use alb.ingress.kubernetes.io/* annotations with NGINX Ingress.
DNS propagation may take several minutes after creating a CNAME record.
Browser issues are often caused by DNS cache or HTTPS redirects.
References
https://kubernetes.github.io/ingress-nginx/
https://docs.aws.amazon.com/eks/
https://helm.sh/docs/






aws configure

aws eks update-kubeconfig --region us-east-1 --name dev-eks-demo

kubectl get nodes

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx   --namespace ingress-nginx   --create-namespace   --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-scheme"="internet-facing"   --set controller.service.type=LoadBalancer

kubectl get ns

kubectl get svc -n ingress-nginx

kubectl get pods --namespace ingress-nginx
   
kubectl get pods -n kube-system | grep ebs

git clone https://github.com/RupaEnggVille/vprofile-eks.git
ls
cd vprofile-eks/
ls
cd manifests/
ls
   
kubectl apply -f .

kubectl get pods

kubectl get deploy

kubectl get svc

kubectl get pvc

kubectl get sc

kubectl get ingress

kubectl describe ingress

kubectl describe ingress vpro-ingress

nslookup vprofile.enggville.xyz

dig vprofile.enggville.xyz

curl http://vprofile.enggville.xyz

curl -H "Host: vprofile.enggville.xyz" http://a0cf17efd9d45406aa7c3b1b806d5365-603018940.us-east-1.elb.amazonaws.com

helm uninstall ingress-nginx -n ingress-nginx

kubectl delete namespace ingress-nginx

kubectl get all -n ingress-nginx

kubectl get svc -A






history 

aws configure

aws eks update-kubeconfig --region us-east-1 --name dev-eks-demo

kubectl get nodes

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml

kubectl annotate svc ingress-nginx-controller -n ingress-nginx service.beta.kubernetes.io/aws-load-balancer-scheme=internet-facing --overwrite

  #kubectl get deployment -n kube-system
  
git clone https://github.com/RupaEnggVille/vprofile-eks.git

ls

cd vprofile-eks/

ls

cd manifests/

ls

kubectl apply -f dbpvc.yaml

kubectl apply -f .

kubectl get ingress
   
kubectl describe ingress vpro-ingress
kubectl get svc -A

Debugging:
  #nslookup vprofile.enggville.xyz
  #dig vprofile.enggville.xyz
  #curl -H "Host: vprofile.enggville.xyz" http://k8s-ingressn-ingressn-883463b903-d4c44cbee2c46232.elb.us-east-1.amazonaws.com
  #kubectl get pods -A | grep ingress
  #kubectl get svc -A | grep ingress
  #kubectl get svc vproapp-service
  #kubectl get endpoints vproapp-service
  #curl http://vprofile.enggville.xyz
  #kubectl get svc -n ingress-nginx
  #kubectl describe ingress vpro-ingress
  kubectl get pvc
   
   kubectl describe pvc
   
   kubectl get sc

kubectl get pods -n kube-system | grep ebs
  Deletion commands:

  kubectl delete svc ingress-nginx-controller -n ingress-nginx
  
  kubectl delete pvc db-pv-claim
   
  

   kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml

   kubectl delete -f .

   
