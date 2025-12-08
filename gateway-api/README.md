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
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/experimental-install.yaml
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
  
