### RBAC ####
(prerequisites : docker and k8s cluster is up and runnig)
Step-by-Step:

Step1 Create Service Accounts
For each team, create a dedicated ServiceAccount in a specific namespace (or kube-system if you want broader control):

(we can create svc account either way , yaml/imperative command , using both one by one)
# For junior devops team
```
cat <<EOF > junior-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: junior-devops
  namespace: kube-system
EOF

```

# For senior devops team

```
kubectl create serviceaccount senior-devops -n kube-system
```

<img width="923" height="378" alt="image" src="https://github.com/user-attachments/assets/ba04b7c2-4a51-404c-9724-fa3490eea79a" />


Step2 Define Roles or ClusterRoles

Junior DevOps (List pods in all namespaces):

```
cat <<EOF > junior-clusterRole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: junior-devops-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get"]
EOF
```

Senior DevOps (Edit all resources but no deletion):

```
cat <<EOF > senior-clusterRole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: senior-devops-role
rules:
- apiGroups: ["", "apps", "batch", "extensions"]
  resources: ["*"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
EOF
```

<img width="850" height="535" alt="image" src="https://github.com/user-attachments/assets/3c59d68a-a1e2-421f-8d8a-ee77750b0914" />


3. Bind the Roles

# Junior
```
kubectl create clusterrolebinding junior-devops-binding \
  --clusterrole=junior-devops-role \
  --serviceaccount=kube-system:junior-devops
```

# Senior
```
cat <<EOF > senior-clusterRoleBinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: senior-devops-binding
subjects:
- kind: ServiceAccount
  name: senior-devops
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: senior-devops-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

<img width="941" height="391" alt="image" src="https://github.com/user-attachments/assets/834e4959-e61b-4a5b-9dea-c7e99cfcbcab" />

  
4. Generate Kubeconfig Files for Them
You can generate kubeconfig for each ServiceAccount using this script or tool like kubeconfig-generator:

✅ Using kubeconfig-generator tool
This tool:
Takes your ServiceAccount and cluster info
Generates kubeconfig with embedded certs
Handles token extraction, base64, etc.

Instllation of kubeconfig-generator tool

```
brew install go
```

```
go install github.com/clockworksoul/k8s-kubeconfig-generator@latest
If ~/go/bin isn’t in your PATH, add it:
```
echo 'export PATH=$PATH:$HOME/go/bin' >> ~/.zshrc
source ~/.zshrc
```

Or for bash:
```
echo 'export PATH=$PATH:$HOME/go/bin' >> ~/.bash_profile
source ~/.bash_profile
```

```
k8s-kubeconfig-generator --help
```

```
kubeconfig-generator --namespace kube-system --service-account junior-devops > junior-kubeconfig
```
It’s open-source, easy to audit, and great for CI/CD provisioning.


🚫 Avoid:
Sharing admin kubeconfig

Embedding cluster-admin tokens in user configs


🔐 Best Practice Summary
Method	Security	Ease	Best For
kubectl config set-*	✅ High	✅ Easy	Manual user/service kubeconfigs
kubeconfig-generator	✅ High	✅ Easy	Automation, CI/CD, service users
OIDC + kubelogin	✅✅ Best	❌ Complex	Large teams, enterprise auth flow



🔐 Summary
Team	Permissions	Method	Kubeconfig Shared
Junior DevOps	list and get pods only	ClusterRole + ServiceAccount	junior-kubeconfig
Senior DevOps	Edit (no delete) all	ClusterRole + ServiceAccount	senior-kubeconfig
