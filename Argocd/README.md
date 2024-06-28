**Lets Talk about argoCD secure deployment**

Today we are going to deploy Helm chart present in git repository from ArgoCD and watch and sync/autosync with cli

Topics covered:

**Simple Helm chart creation for public Image and push to Github repo**
**deploy Argocd on any managed service (GKE,EKS,AKS) or in any self managed cluster(kind,minikube)**
**Deploy nginx ingress controller and Cert manager and expose ArgoCD with Ingress and Cert manager**
**Access ArgoCD webUI**
**Added Github repo in secure way**
**Create apps of apps file to integrate ArgoCD with github repository**
**Install argocd Cli locally and list all the running applications**


Step 1: 
**create nginx namespace and create helm chart**

kubectl create ns nginx
helm create <chart name>

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/4ca45ead-022c-4403-8466-6af8a70aa265)

Step 2:

Make changes in the values.yaml file accordin to requirenment 
