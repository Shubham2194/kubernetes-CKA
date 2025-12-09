###Gateway API is the next-generation traffic routing API for Kubernetes, designed to replace or extend the old Ingress system ###

Prerequisites:
- We need a Kubernetes cluster

Main components:
- Gateway Class - Type of load balancing implementation
- Gateway - Actual Load balamcer instance
- Http Route - Routing rules for mapping traffic

<img width="957" height="317" alt="image" src="https://github.com/user-attachments/assets/12c9e525-29a9-4b31-bb80-ad08dabd0a5d" />


Step 1: 
Install Gateway API CRDs

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/latest/download/standard-install.yaml
```

<img width="1238" height="304" alt="image" src="https://github.com/user-attachments/assets/9c4cb2c2-e30f-4e9d-96cb-2aa1bbfef9ef" />


This will install (Stable Channel):

Gateway Classes: kubectl get gatewayclass
Gateways: kubectl get gateway
HTTP Routes: kubectl get httproute
These APIs are part of the Experimental Channel:

TLS Routes: kubectl get tlsroute
TCP Routes: kubectl get tcproute
UDP Routes: kubectl get udproute

Note: Gateway API is very new, and all of the above is subject to change quite rapidly
  


Step 2:
Install NGINX Gateway

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/standard-install.yaml
helm install ngf oci://ghcr.io/nginxinc/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway


```

<img width="1236" height="428" alt="image" src="https://github.com/user-attachments/assets/cf1e9660-3151-4a3d-9361-af679e7e5203" />


Step 3:
Verify the Deployment and Access the Gateway

```
kubectl get all -n nginx-gateway
```
<img width="1229" height="565" alt="image" src="https://github.com/user-attachments/assets/23179a93-ba3d-4ced-822a-b8374eff82ed" />


Once the EXTERNAL-IP is available, you can retrieve it with this command:

```
export GW_URL=$(kubectl get svc ngf -n nginx-gateway -o jsonpath='{.status.loadBalancer.ingress[].ip}')
echo "NGINX Gateway URL: http://$GW_URL/"
```

Step 4:
Create the Gateway Resource

```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: ngf-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
```

```
kubectl apply -f gateway.yaml
kubectl get gateway -A
```

<img width="763" height="144" alt="image" src="https://github.com/user-attachments/assets/aaf6d164-9187-44a2-80b0-ed58bdc66fe1" />

✅ STEP 5:
Now apply your HTTPRoute

```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: backend-route
  namespace: backend # Place the route in the same namespace as your service
spec:
  parentRefs:
  - name: ngf-nginx-gateway-fabric # Name of the Gateway resource you created
    namespace: nginx-gateway # Namespace where your Gateway is deployed (adjust if different)
  hostnames:
  - "" # Replace with your actual domain or leave blank to match all hostnames
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: xyz # Name of your backend Service from the image
      port: 8000 # Port of your backend Service from the image
```

