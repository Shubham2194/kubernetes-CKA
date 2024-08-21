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

cd /home/antino/Downloads/Tlabs/flipspaces/fs-repos/fs.apis
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

#sudo docker build --no-cache -t chat . --build-arg APP_NAME=chat
docker build  --no-cache -t ${registry} --build-arg APP_NAME=${APP_NAME} .
aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${awsAccountId}.dkr.ecr.${awsRegion}.amazonaws.com
docker tag ${registry}:latest ${imageRepository}:${imageTag}
docker push ${imageRepository}:${imageTag}

```

Step 3:
Create a Helm chart in EKS and deploy the API manually

 - helm create api
  
