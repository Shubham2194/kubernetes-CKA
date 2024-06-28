**Lets Talk about argoCD secure deployment**

Today we are going to deploy Helm chart present in git repository from ArgoCD and watch and sync/autosync with cli

Topics covered:

**1) Simple Helm chart creation for public Image and push to Github repo**

**2) deploy Argocd on any managed service (GKE,EKS,AKS) or in any self managed cluster(kind,minikube)**

**3) Deploy nginx ingress controller and Cert manager and expose ArgoCD with Ingress and Cert manager**

**4) Access ArgoCD webUI**

**5) Added Github repo in secure way**

**6) Create apps of apps file to integrate ArgoCD with github repository**

**7) Install argocd Cli locally and list all the running applications**


Step 1: 
**create nginx namespace and create helm chart**

kubectl create ns nginx
helm create <chart name>

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/4ca45ead-022c-4403-8466-6af8a70aa265)

Step 2:
**Make changes in the values.yaml file accordin to requirenment**

we are keeping image as nginx with latest tag

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/08b19f73-85fe-476b-8a73-396ff1cbb903)


Step 3: 

**Now push helm chart on github and deploy argocd in k8s envirment**

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/041afcc6-f9c4-48fa-8be3-1a45508c65e3)


kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


kubectl get pods,svc -n argocd

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/1f06e5a3-8eea-44e6-b027-6a017a9b8d43)

Step 4:

**deploy nginx ingress cotroller and Cert-manager in K8s environment**

kubectl create namespace ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

kubectl get pods --namespace ingress-nginx


kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.yaml

kubectl -n cert-manager get all

Step 5:
**Expose ArgoCD with nginx ingress resource**

Find the ingress.yaml and cert.yaml in the root directory 
kubectl apply -f cert.yaml -f ingress.yaml

kubectl get ing,cert -n argocd


Step 6:
**Deploy helm chart in the Nginx namespace from argocd using apps of apps and sync using argocd cli**



















