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


**COOL!**
