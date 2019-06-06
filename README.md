# GatewayProject-k8s

a continuation of GatewayProject (at: https://github.com/SamVivers/GatewayProject). This version should be deployed using kubernetes

This project consists of several APIs (Application programming interfaces) which provide a secure login, with email verification, to a dashboard where projects may be accessed.

![alt text](https://raw.githubusercontent.com/SamVivers/images/master/MicroservicesSchema.jpg)
where the arrows indicate dirrection of dependance (i.e. mongo-service depends on mongo)

# Prerequisites

access to GCP (could be build on other cloud services, but the setup will use GCP) 

# Setup

open the GCP cloud shell

kick up a kubernetes cluster
``` 
gcloud container clusters create [CLUSTER_NAME] --zone [COMPUTE_ZONE]
```
authenticate kubectl for the cluster
```
gcloud container clusters get-credentials [CLUSTER_NAME]
```
clone this git repo
```
git clone https://github.com/SamVivers/GatewayProject-k8s.git
```
run the microservices
``` 
kubectl apply -f deployments/
kubectl apply -f services/
kubectl apply -f gateway/
```
once the gateway's load balancer is up it will be assigned an IP address and this must be put into the authentication-service-deployment.yaml
``` 
cd deployments/
vim authentication-service-deployment.yaml
```
and replace my IP address in the ACTIVATION_LINK environment variable with the one generated for you
![alt text](https://raw.githubusercontent.com/SamVivers/images/master/authentication-service-deployment.jpg)

then restart the deployments
```
cd ../
kubectl apply -f deployments/
```
in a browser head to http://"your IP"/authentication/register and sign up

activate your account via the auto-sent email

head to http://"your IP"/authentication/login and login

# CI with Jenkins

In order to automate updating first start jenkins, in the cloud shell
```
kubectl apply -f jenkins/
```
Head to the Jenkins load balancer IP address and sign in, initial admin password can be found in the jenkins pod logs
```
kubectl logs [JENKINS_POD_NAME]
```
After the initial setup, create a job to which will clone the dev branch of the GatewayProject repo, build docker images, push them and apply these images. The shell command will look something like this
```
docker-compose build
docker-compose push
kubectl --record deployment.apps/[KUBERNETES_SERVICE] set image deployment.v1.apps/[KUBERNETES_SERVICE] [KUBERNETES_SERVICE]=docker.io/[YOUR_DOCKER.IO_ACCOUNT]/[KUBERNETES_SERVICE]:${BUILD_NUMBER}
```
Here each of the 13 kubernetes services should be updated (not mongo as it is the default image)

Changes must be made to the docker-compose.yaml cloned from GatewayProject as it currently tags images with my docker.io account. You must also login into your docker.io account before pushing images, I would recomment doing this from the from inside the jenkins pod (it could alternatively be done as part of the Jenkins job)
```
kubectl exec -it [JENKINS_POD_NAME] bash
```
then
```
docker login
```
Tick the box on Jenkins allowing GitHub WebHooks, this is already configured in the GatewayProject repo, and when changes to the source code are made your Jenkins job will build, creating and deploying new docker images to the kubernetes services

![alt text](https://raw.githubusercontent.com/SamVivers/images/master/CI-Lifecycle.jpg)
