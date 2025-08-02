# 🧠 EFK Logging Stack for EKS with ARM64 (Graviton) Node Support

(Checkout all the files in the repo for refrence)

This repository contains a **production-ready logging stack using EFK (Elasticsearch, Fluentd, Kibana)** for an Amazon EKS cluster running on **ARM64 (Graviton)** nodes.

✅ Fluentd is deployed as a **custom Docker image** built for ARM64.  
✅ Tolerations are applied to ensure Fluentd runs on every Graviton node.  
✅ Kibana is secured via **Nginx reverse proxy** (as a sidecar container).  
✅ Kibana + Nginx is exposed using the **AWS ALB Ingress Controller**.

---

## 🔧 Step-by-Step Setup , make sure to Clone the repo and make all the below changes

### 1. 📦 Build Custom Fluentd Docker Image for ARM64

Official Fluentd images don't fully support ARM64 architecture. So we built our own image:

```Dockerfile
# Start with your existing ARM64 Fluentd image
FROM arm64v8/fluentd:v1.18-debian-1

# Switch to root user to install the build dependencies and gems
USER root

# Install build dependencies for the native gem extensions.
RUN apt-get update && \
    apt-get install -y --no-install-recommends build-essential ruby-dev && \
    rm -rf /var/lib/apt/lists/*

# Install all required Fluentd plugins
RUN gem install fluent-plugin-kubernetes_metadata_filter --no-document && \
    gem install fluent-plugin-elasticsearch --no-document && \
    gem install fluent-plugin-multi-format-parser --no-document

# Clean up build dependencies to reduce image size
RUN apt-get purge -y --auto-remove build-essential ruby-dev

# Switch back to the non-root fluent user for security
USER fluent
```

2. 📤 Push Image to Amazon ECR

```
# Authenticate to ECR
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# Tag and push
docker build -t fluentd-arm64 .
docker tag fluentd-arm64:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/fluentd-arm64:latest
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/fluentd-arm64:latest
```



3. 📜 Use Custom Image in Fluentd DaemonSet Manifest
In your Fluentd DaemonSet manifest:
```
containers:
  - name: fluentd
    image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/fluentd-arm64:latest
    ...

```

4. 🎯 Add Tolerations to Schedule Fluentd on ARM Nodes
   
Ensure that your Graviton nodes are tainted (e.g., internal=arm64:NoSchedule), then add tolerations in the DaemonSet:

```
tolerations:
  - key: "internal"
    operator: "Equal"
    value: "arm64"
    effect: "NoSchedule"
```

5. 🔐 Add Nginx as a Reverse Proxy in the Kibana Pod
To protect Kibana access, use a multi-container pod setup with Nginx + Kibana:

```
  containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: auth
          mountPath: /etc/nginx/auth

      volumes:
      - name: nginx-config
        configMap:
          name: kibana-nginx-config
      - name: auth
        secret:
          secretName: kibana-auth-secret

```

And create a ConfigMap with your Nginx config:
```
# nginx.conf
server {
    listen 8080;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/auth/auth;
    location / {
        proxy_pass http://localhost:5601;
    }
}

```

6. 🌐 Expose Kibana Securely via AWS ALB Ingress Controller

 ```

  apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kibana-ingress
  namespace: logging
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: 'ip'
    alb.ingress.kubernetes.io/load-balancer-name: <lb name>
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/certificate-arn: acm cert arn>
    alb.ingress.kubernetes.io/group.name: <ingress-group-name>
    alb.ingress.kubernetes.io/subnets: <subnets>

spec:
  ingressClassName: alb
  rules:
  - host: <your host name>
    http:
      paths:
      - backend:
          service:
            name: kibana
            port:
              number: 8080
        path: /
        pathType: Prefix
```

7. After making all the changes in the daemonset of fluentd, nginx cm and ingress controller , apply it
   
```
 kubectl apply -f kubernetes-CKA/logging-eks .
```

<img width="755" height="296" alt="image" src="https://github.com/user-attachments/assets/b9ed9f1d-359e-438f-b328-08468c0dcdf3" />

8. Check your ingrss is mapped with ALb and available

```
kubectl get ing -n logging
```

9. Lastly hit your kibana host in Browser and fill you auth creds to access kibana dashboard
Note: make sure to map it with your DNS (route53 , cloudflare , hostinger etc)

now goto :
home > index_management > index patterns > type logstash-* and save it 
Now goto home > discover and in search type:
```
kubernetes.namespace_name : backend
```

## WE are Done and Setuped Centralized Logging !!!





