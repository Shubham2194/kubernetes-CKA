### RBAC ####
(prerequisites : docker and k8s cluster is up and runnig)

Step-by-Step:

( I am Using KIND Cluster for this POC )

Step1: Create Service Accounts and Token

For each team, create a dedicated ServiceAccount in a specific namespace (or kube-system if you want broader control):

(we can create resources either way , yaml/imperative command)
# For junior devops team
```
cat <<EOF > junior-serviceaccount.yaml
# junior-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: junior-devops
  namespace: kube-system

---

# sa-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: junior-devops-token
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "junior-devops"
type: kubernetes.io/service-account-token
EOF

```

# For senior devops team

```
# senior-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: senior-devops
  namespace: kube-system

---

# sa-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: senior-devops-token
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "senior-devops"
type: kubernetes.io/service-account-token
```


<img width="1002" height="280" alt="image" src="https://github.com/user-attachments/assets/f439c719-2f44-40d0-b299-92d02d8d5dc6" />



Step2: Define ClusterRoles

Junior DevOps (List pods in all namespaces):

```
cat <<EOF > junior-clusterRole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: junior-devops-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods", "services", "namespaces", "configmaps", "secrets", "nodes", "events"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch"]
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
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets", "endpoints", "persistentvolumeclaims", "namespaces", "events"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses", "networkpolicies"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings"]
  verbs: ["get", "list", "watch"]
EOF
```


<img width="923" height="212" alt="image" src="https://github.com/user-attachments/assets/98b5c4d5-2caa-430c-8a49-814f05abf686" />




Step 3: Bind the Cluster Roles

# Junior
```
cat <<EOF > junior-clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: junior-devops-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: junior-devops
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: junior-devops-clusterrole
  apiGroup: rbac.authorization.k8s.io
EOF
```

# Senior
```
cat <<EOF > senior-clusterRoleBinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: senior-devops-clusterrolebinding
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


<img width="1244" height="215" alt="image" src="https://github.com/user-attachments/assets/443133e7-ecf9-4ec4-9b6e-00436bedba44" />

  
Step 4. Generate Kubeconfig Files for Them
You can generate kubeconfig for each ServiceAccount using this script:

✅ 
```
#!/bin/bash
# Usage: ./generate-kubeconfig.sh <SERVICE_ACCOUNT_NAME> <NAMESPACE>
SERVICE_ACCOUNT=$1
NAMESPACE=$2
OUTPUT_FILE="${SERVICE_ACCOUNT}.kubeconfig"
SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

# === Find Secret Name for the Service Account ===
SECRET_NAME=$(kubectl -n "$NAMESPACE" get secret | grep "$SERVICE_ACCOUNT" | awk '{print $1}')

if [ -z "$SECRET_NAME" ]; then
  echo "❌ Secret for service account '$SERVICE_ACCOUNT' not found in namespace '$NAMESPACE'"
  exit 1
fi

echo "✅ Found secret: $SECRET_NAME"

# === Extract credentials ===
CA=$(kubectl -n "$NAMESPACE" get secret "$SECRET_NAME" -o jsonpath='{.data.ca\.crt}')
TOKEN=$(kubectl -n "$NAMESPACE" get secret "$SECRET_NAME" -o jsonpath='{.data.token}' | base64 --decode)
NS=$(kubectl -n "$NAMESPACE" get secret "$SECRET_NAME" -o jsonpath='{.data.namespace}' | base64 --decode)

# === Generate kubeconfig ===
cat <<EOF > "$OUTPUT_FILE"
apiVersion: v1
kind: Config
clusters:
- name: kind-cluster
  cluster:
    certificate-authority-data: ${CA}
    server: ${SERVER}
contexts:
- name: ${SERVICE_ACCOUNT}-context
  context:
    cluster: kind-cluster
    namespace: ${NS}
    user: ${SERVICE_ACCOUNT}-user
current-context: ${SERVICE_ACCOUNT}-context
users:
- name: ${SERVICE_ACCOUNT}-user
  user:
    token: ${TOKEN}
EOF

echo "✅ Kubeconfig written to: $OUTPUT_FILE"
echo "👉 Use with: KUBECONFIG=$OUTPUT_FILE kubectl get pods --all-namespaces"
```


<img width="1672" height="868" alt="image" src="https://github.com/user-attachments/assets/47730fa1-a8ba-4d88-ab1e-48788f4583d1" />


Step 5 : Copy the kubeconfig file add in your Local path i.e ~/.kube/config , i am using lens here to add my kubeConfig of junior devops team

Try editing you pod in any namespace , i have nginx pod in backend namespace (tried increasing replica's to 2 ) 
So as per the ClusterRole it should not be able to edit/delete the resources and i am experiencing the same.

<img width="1166" height="853" alt="image" src="https://github.com/user-attachments/assets/21ed0764-da80-4137-8929-aaeafcc4751f" />

Step 6: Check if both kubeconfig have appropriate access 

```
 kubectl auth can-i delete pods --as=system:serviceaccount:kube-system:junior-devops -n backend
```

```
 kubectl auth can-i delete pods --as=system:serviceaccount:kube-system:senior-devops -n backend
```

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
Senior DevOps	Edit,view,list (no delete) all	ClusterRole + ServiceAccount	senior-kubeconfig
