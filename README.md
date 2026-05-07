# vprofile-eks

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

   
