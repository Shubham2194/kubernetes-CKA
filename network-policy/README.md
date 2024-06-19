![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/55130110-90d8-41a7-829b-e8fa63788c12)


**Step 1: Label Your Namespaces**

First, label your namespaces to make them identifiable in your network policies.

**Backend Namespace:**

apiVersion: v1
kind: Namespace
metadata:
  name: backend
  labels:
    name: backend


**Database Namespace:**

apiVersion: v1
kind: Namespace
metadata:
  name: database
  labels:
    name: database


kubectl apply -f backend-namespace.yaml
kubectl apply -f database-namespace.yaml


**Step 2: Create the Network Policy in the Database Namespace**

Next, create a Network Policy that allows ingress traffic to the Database namespace only from the Backend namespace.

**Network Policy YAML:**
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-database
  namespace: database
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: backend


kubectl apply -f allow-backend-to-database.yaml


![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/b3662fa2-783b-438f-82aa-af47a9680ba2)



**COOL!**
