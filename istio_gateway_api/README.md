###Introduction###
The Kubernetes Gateway API represents the future of ingress and service mesh traffic management. 
Unlike traditional Ingress controllers, Gateway API provides a more expressive, extensible, 
and role-oriented approach to configuring network access to your services.

In this comprehensive tutorial, we’ll walk through setting up Istio with Gateway API, deploying a sample application, 
and configuring both HTTP and HTTPS routing with proper TLS certificates. By the end, you’ll have a production-ready 
setup with namespace isolation and security best practices.

What You’ll Build
✅ Istio service mesh with Gateway API support
✅ Multi-namespace architecture (apps, gateway, certs)
✅ NGINX application deployment
✅ HTTP and HTTPS routing with custom hostnames
✅ TLS certificate management with cert-manager
✅ Cross-namespace access control with ReferenceGrants

Prerequisites
Before starting, ensure you have:

A running Kubernetes cluster (Docker Desktop, kind, EKS, or minikube)
kubectl CLI installed and configured
Basic understanding of Kubernetes concepts (pods, services, deployments)

Let's Begin:
Part 1: Installing Istio and Gateway API

Step 1: Download and Install Istio

```
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.28.0 sh -
```

```
# Add istioctl to your PATH
cd istio-1.28.0
export PATH=$PWD/bin:$PATH
```

```
#Install Istio with the demo profile, which includes all necessary components:

istioctl install --set profile=demo -y
```

What this does:

Installs Istio control plane (istiod) in the istio-system namespace
Deploys ingress and egress gateways
Configures Istio with settings suitable for learning and development


```
#Verify the installation:

kubectl get pods -n istio-system
```

<img width="1023" height="493" alt="image" src="https://github.com/user-attachments/assets/4e27092c-6e3e-41b7-9f26-212ff2010100" />



Step 2: Install Gateway API CRDs

The Gateway API resources (Gateway, HTTPRoute, etc.) are not part of core Kubernetes yet, so we need to install the Custom Resource Definitions:

```
kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd" | kubectl apply -f -
```

Verify the Gateway API resources are available:

```
kubectl api-resources | grep gateway
```

Part 2: Setting Up the Namespace Architecture

A well-structured namespace architecture improves security and organization. We’ll create three namespaces:

namespace.yml:

```
apiVersion: v1
kind: Namespace
metadata:
  name: backend
  labels:
    name: backend
---
apiVersion: v1
kind: Namespace
metadata:
  name: gateway
  labels:
    name: gateway
---
apiVersion: v1
kind: Namespace
metadata:
  name: certs
  labels:
    name: certs
    purpose: tls-certificates
```

Apply the namespaces:

```
kubectl apply -f namespace.yml
```

Why three namespaces?

backend: For application workloads
gateway: For ingress gateways (isolation from apps)
certs: For centralized TLS certificate storage

Part 3: Deploying the Application

Step 1: Create the Deployment
deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: backend
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---

apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: backend
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

Apply:
```
kubectl apply -f deployment.yaml

#Verify the deployment:

kubectl get pods,svc -n backend
```

Part 4: Configuring TLS Certificates
For HTTPS support, we need TLS certificates. We’ll use cert-manager to generate self-signed certificates.

Step 1: Install cert-manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```


Wait for cert-manager pods to be ready:
```
kubectl get pods -n cert-manager
```

<img width="756" height="257" alt="image" src="https://github.com/user-attachments/assets/6dbaa0c0-c6db-4600-a50a-532560e7d535" />

Step 2: Create a ClusterIssuer and Certificate

clusterissuer.yml:
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: ca-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-local-cert
  namespace: certs
spec:
  secretName: app-local-tls
  issuerRef:
    kind: ClusterIssuer
    name: ca-issuer
  dnsNames:
  - "*.xyz.ai"
```


Apply it:
```
kubectl apply -f clusterissuer.yml
#Verify the certificate was created:

kubectl get certificate -n certs
kubectl get secret app-local-tls -n certs
```



Part 5: Creating the Gateway

The Gateway resource defines how external traffic enters the cluster.

gateway.yml:

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gateway
  namespace: gateway
spec:
  gatewayClassName: istio
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: Selector
          selector:
            matchLabels:
              name: backend   # <-- Allow routes from 'backend' namespace
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        mode: Terminate
        certificateRefs:
        - name: app-local-tls
          namespace: certs
      allowedRoutes:
        namespaces:
          from: Selector
          selector:
            matchLabels:
              name: backend   # <-- Allow routes from 'backend' namespace
```


Key points:

gatewayClassName: istio tells Kubernetes to use Istio's Gateway implementation
We define two listeners: one for HTTP (port 80) and one for HTTPS (port 443)
The HTTPS listener references our TLS certificate from the certs namespace
allowedRoutes.from: Selectorallows routes from certain labeled namespace.
Apply the gateway:

```
kubectl apply -f gateway.yml
```

Check the gateway status:

```
kubectl get gateway -n gateway
```















