# vprofile-eks

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

   
