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
###### gcloud container clusters create [CLUSTER_NAME] --zone [COMPUTE_ZONE]

authenticate kubectl for the cluster
###### gcloud container clusters get-credentials [CLUSTER_NAME]

clone this git repo
###### git clone https://github.com/SamVivers/GatewayProject-k8s.git

run the microservices
###### kubectl apply -f deployments/
###### kubectl apply -f services/
###### kubectl apply -f gateway/

one the gateway's load balancer is up it will be assigned an IP address and this must be put into the authentication-service-deployment.yaml
###### cd deployments/
###### vim authentication-service-deployment.yaml

and replace my IP address with the one generated for you

then restart the deployments
###### cd ../
###### kubectl apply -f deployments/

in a browser head to http://"your IP"/authentication/register and sign up

activate your account via the auto-sent email

head to http://"your IP"/authentication/login and login
