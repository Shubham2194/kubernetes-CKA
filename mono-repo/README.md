![image](https://github.com/user-attachments/assets/2469002f-37c2-42e9-974e-87f952fbf90b)



A **monorepo** is a way of organizing code where all related projects or parts of a software system are kept in a single repository (a "repo"). 
Instead of having separate repos for different projects, everything is stored together in one place.

So this is the structure , in the root directory we have apps folder inside which we have all the microservices directory present:

![image](https://github.com/user-attachments/assets/23f7b39f-15be-428d-b750-026f3f584c00)


Step1:
Create a Dockerfile on the root level which communicate with all the api's

```Dockerfile
FROM node:18.18.0

ARG APP_NAME
ENV APP_NAME ${APP_NAME}

WORKDIR /app

RUN apt-get -qy update && apt-get -qy install openssl

COPY package*.json .npmrc .env ./

RUN npm install -g nx
RUN npm install

COPY . .

EXPOSE 3333

RUN nx reset

CMD nx serve ${APP_NAME}
```
By using Argument we are creating docker images for all the microservices with single Dockerfile

Step 2:
Create a bash script which will take APP_NAME and Image name as argument and create new Dockerimage and push to ECR

```bash
#!/bin/bash

git pull origin develop

# Check if the required arguments are provided
if [ $# -lt 2 ]; then
    echo "Usage: $0 <APP_NAME> <imageTag>"
    exit 1
fi

# Assign arguments to variables
APP_NAME=$1
imageTag=$2

imageRepository="44421XXXXX.dkr.ecr.ap-south-1.amazonaws.com/fs-dev"
registry="abc" #your ECR registry name
awsRegion="ap-south-1"
namespace="backend"
awsAccountId="44421XXXXXX"

docker build  --no-cache -t ${registry} --build-arg APP_NAME=${APP_NAME} .
aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${awsAccountId}.dkr.ecr.${awsRegion}.amazonaws.com
docker tag ${registry}:latest ${imageRepository}:${imageTag}
docker push ${imageRepository}:${imageTag}

```

Step 3:
Create a Helm chart in EKS and deploy the API manually

 - aws eks update-kubeconfig --region ap-south-1 --name abc
 - helm create api
 - nano api/values.yaml and change the following:

```yml 
replicaCount: 1

image:
  repository: 44421xxxx.dkr.ecr.ap-south-1.amazonaws.com/abc #URL of your ECR
  pullPolicy: IfNotPresent
  tag: "api"   #Image TAG

imagePullSecrets:
- name:  ecr-registry-secret #ECR pull secrets 

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""

podAnnotations: {}
podLabels: {}
service:
  type: ClusterIP
  port: 80
  targetport: 3333 #API port
```

Step 4:

ADD the Port in deployment.yaml - Change to {{ .Values.service.targetport }}


```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "chat.fullname" . }}
  labels:
    {{- include "chat.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "chat.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "chat.labels" . | nindent 8 }}
        {{- with .Values.podLabels }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "chat.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetport }}
              protocol: TCP
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- with .Values.volumeMounts }}
          volumeMounts:
            {{- toYaml . | nindent 12 }}
          {{- end }}
      {{- with .Values.volumes }}
      volumes:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

Step 5: Changes TargetPort value in service.yml

```yml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "chat.fullname" . }}
  labels:
    {{- include "chat.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetport }}
      protocol: TCP
      name: http
  selector:
    {{- include "chat.selectorLabels" . | nindent 4 }}
```

Step 6:

Helm upgrade --install api api

And this is how we can manage Mono-repo deployment with single Dockerfile , we can Automate this as well using Jenkins or any CI/CD tool.

**We have Successfully deployed our Mono Repo API in the EKS !!**



  
