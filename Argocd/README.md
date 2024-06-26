**Lets Talk about argoCD secure deployment**

Today we are going to deploy Helm chart present in git repository from ArgoCD and watch and sync/autosync with cli

Topics covered:
	**Simple Helm chart creation for public Image and push to Github repo**
        **Run kind or any Managed cluster (GKE,EKS,AKS) and deploy ArgoCD**
        **Deploy nginx ingress controller and Cert manager and expose ArgoCD with Ingress and Cert manager**
        **Access ArgoCD webUI**
        **Added Github repo in secure way**
        **Create apps of apps file to integrate ArgoCD with github repository**
        **Install argocd Cli locally and list all the running applications**

Step 1: 
helm create <chart name>
