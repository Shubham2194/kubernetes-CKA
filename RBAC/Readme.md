### RBAC ####

Step-by-Step:

Step1 Create Service Accounts
For each team, create a dedicated ServiceAccount in a specific namespace (or kube-system if you want broader control):

(we can create svc account either way , yaml/imperative command , using both one by one)
# For junior devops team
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: junior-devops
  namespace: kube-system

```

# For senior devops team

```
kubectl create serviceaccount senior-devops -n kube-system
```

Step2 Define Roles or ClusterRoles

Junior DevOps (List pods in all namespaces):

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: junior-devops-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get"]
```

Senior DevOps (Edit all resources but no deletion):

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: senior-devops-role
rules:
- apiGroups: ["", "apps", "batch", "extensions"]
  resources: ["*"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
```

3. Bind the Roles

# Junior
```
kubectl create clusterrolebinding junior-devops-binding \
  --clusterrole=junior-devops-role \
  --serviceaccount=kube-system:junior-devops
```

# Senior
```
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
```
  
4. Generate Kubeconfig Files for Them
You can generate kubeconfig for each ServiceAccount using this script or tool like kubeconfig-generator:

✅ Using kubeconfig-generator tool
This tool:
Takes your ServiceAccount and cluster info
Generates kubeconfig with embedded certs
Handles token extraction, base64, etc.

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
