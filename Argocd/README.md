ArgoCD 
**Let's get stuff deployed!**

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/2cc743c1-3f38-4341-8c21-9533c0617664)



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

**Now push helm chart on github and deploy Argocd in k8s envirment**

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/041afcc6-f9c4-48fa-8be3-1a45508c65e3)


kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Get username and password for **ArgoCD**

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
Default Username: admin
(save these creds , we will use while accessing/login argoCD)


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

**Access ArgoCD and Install argocd Cli**
Access ArgoCD UI from the Host 

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/44f26d4f-eba2-4feb-a120-b713baa9605f)

Install Argocd CLI (Ubuntu)(if you are on mach on windows : https://argo-cd.readthedocs.io/en/stable/cli_installation/ )

curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

**LOGIN TO ARGO**
argocd login argo.tlabs.ai --grpc-web


![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/ef238452-c795-46df-ba89-c993a10164d2)


Step 7:
**Deploy Nginx Helm chart in Nginx namespace with apps of apps and list app with argoCD cli**

Find the argocd-apps.yaml in the root directory

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/9fe24812-a61c-494b-880a-acbc2d7c211d)


Change the repoURL accordingly and Path of your Chart path.

Kubectl apply -f argocd-apps.yaml

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/48440778-e01e-477c-819a-8103629649f4)

kubectl get pods,svc -n nginx

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/979dc850-30af-4e55-b1f5-0edce8797afa)


step 8:
**Check Argocd applications list using cli and sync**

 argocd app get nginx
 
![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/482240b2-f9b7-4a51-b7c4-1b40c0cdcb2b)

Check all the apps running

argocd app list

Sync new tag using argoCD cli

argocd app sync nginx

If everything is successful, you'll see an output like the following one

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/d052ddd6-fdee-40e1-b3f0-3212b037ff66)



***Congratulations! We've successfully deployed our first application successfully using ArgoCD. _I'm pretty sure, that took less than 15 minutes_!***

***The goal of an application like ArgoCD is to ensure that the `current state` and `desired state` remain the same. To see that in action, delete the nginx deployement using kubenctl. Immediately you'll see the `syncstatus` as `OutOfSync`, and it will autosync and deploy nginx helm chart again and soon you will see the service will be recreated.***

***So that was my short and quick getting started guide for ArgoCD. Stay tuned for more and more posts that you'll find helpful :)*** 



![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/8a445524-27c8-4462-9adc-e31f4df3775f)










