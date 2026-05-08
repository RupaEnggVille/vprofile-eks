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

   
