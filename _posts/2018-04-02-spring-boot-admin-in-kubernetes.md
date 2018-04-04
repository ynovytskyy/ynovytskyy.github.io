---
layout: post
title:  "Spring Boot Admin in Kubernetes"
---
[Spring Boot Admin](https://github.com/codecentric/spring-boot-admin) is a
Codecentric's community project that provides an admin interface for [Spring Boot ®](http://projects.spring.io/spring-boot/) applications. To an extent it might serve as the missing tool in environments like Kubernetes or Docker Swarm that are great container orchestration platforms but are missing UIs for insight into services state, monitoring and debugging.

- Prerequisites
  - Tools
  - Set up K8s cluster on GCP
- Approaches to use Spring Boot Admin
- Spring Boot Admin with Eureka
  - Setup Eureka Service Registry
  - Setup Spring Boot Admin
  - Deploy a demo micro-service
- Experimenting

# Prerequisites
We need a Kubernetes cluster, Docker to build our images, Docker repository to push you custom container images to, and to pull from when you deploy.

## Install tools
```bash
#you do need `brew`, if you don't have it yet https://brew.sh
brew tap caskroom/versions
brew cask install java8
brew install kubectl jq
brew cask install docker
open /Applications/Docker.app
brew cask install google-cloud-sdk
```

## Set up Kubernetes cluster on GCP
For Kubernetes cluster and Docker repository we are going to use Google Cloud Platform aka GCP. Check <https://cloud.google.com/free/> for the details of what you get for free for your experiments with GCP.

When initializing with `gcloud init` you will be taken to your web browser to login into your google account and to allow it to use GCP. To check on regions/zones that you might prefer go to <https://cloud.google.com/compute/docs/regions-zones/>
```bash
gcloud init
gcloud config set compute/zone us-east1-b
gcloud config set compute/region us-east1

gcloud projects create cso-ynovytskyy
gcloud config set project cso-ynovytskyy

#For experiments the following should be enough, so saving some $ here. Other wise use defaults: 3 nodes of `n1-standard-1`
gcloud container clusters create kub-cluster -m=f1-micro --num-nodes=2
gcloud container clusters get-credentials kub-cluster

kubectl cluster-info
```
Now ready to roll on Kubernetes level of abstraction with `kubectl` CLI.

# Approaches to use Spring Boot Admin

# Spring Boot Admin with Eureka
Let's go and `git clone` some prepared code, docker image descriptors and k8s manifests for our Spring Boot Admin experiment:
```bash
git clone git@github.com:ynovytskyy/spring-boot-admin-in-kubernetes.git
cd spring-boot-admin-in-kubernetes
```

## Setup Eureka Service Registry
At this point we should be able to deploy Eureka Service Registry to `kub-cluster`
```bash
kubectl apply -f k8s-eureka.yml
kubectl get pod
kubectl get svc
```
Lats two command let us see Pods and Services deployed. I've kept it simple and deployed Eureka as a Pod. Once the pod and service are up and load balancer gets it's external IP address assigned, you can navigate to that IP address with port `8761` and see Spring-flavored Eureka UI. Or use the following fancy quirky Bash command that shows how parse out external IP and navigate to it in a scripted way.
```bash
ip=$(kubectl get svc -o=json | jq -r '.items[] | select(.metadata.name == "eureka").status.loadBalancer.ingress[0].ip'); open "http://$ip:8761"
```
> **Note:** Eureka is exposed only for the purpose of this example.

## Setup Spring Boot Admin
Now we need to build Spring Boot Admin application - Maven project first to produce Spring Boot executable jar, then build a Docker image of it.
```bash
cd spring-boot-admin
./mvnw clean package
docker build -t spring-boot-admin .
cd ..
```
Now we have Docker container image of Spring Boot Admin in our local Docker image repo. Now let's push to it to `GCP Container Registry (GCR)`. Tag your image according to <https://cloud.google.com/container-registry/docs/pushing-and-pulling#choosing_a_registry_name>
```bash
gcloud beta auth configure-docker # in the future the "beta" piece should not be needed

docker tag spring-boot-admin gcr.io/cso-ynovytskyy/spring-boot-admin
gcloud docker -- push gcr.io/cso-ynovytskyy/spring-boot-admin #GCP doc says to use "docker push <imge-tag>" but it didn't work for me
```
So the `spring-boot-admin` is now pushed to Google Container Registry and can be user from Kubernetes manifest.
```bash
#in the root of the repo
kubectl apply -f k8s-spring-boot-admin.yml
kubeclt get svc
#when service's ready use external IP and port 8080 or...
ip=$(kubectl get svc -o=json | jq -r '.items[] | select(.metadata.name == "admin").status.loadBalancer.ingress[0].ip'); open "http://$ip:8080"
```

## Deploy a demo micro-service
It's time to deploy a demo micro-service application and see how Spring Boot Admin helps to get information about its instances.
```bash
cd demo-micro-service
./mvnw clean package
docker build -t demo-micro-service .
cd ..
docker tag demo-micro-service gcr.io/cso-ynovytskyy/demo-micro-service
gcloud docker -- push gcr.io/cso-ynovytskyy/demo-micro-service

kubectl apply -f k8s-spring-boot-admin.yml
kubeclt get svc
#when service's ready use external IP and port 8080 or...
ip=$(kubectl get svc -o=json | jq -r '.items[] | select(.metadata.name == "demo-micro-service").status.loadBalancer.ingress[0].ip'); open "http://$ip:8080"
```

# Experimenting
Open Spring Boot Admin and check that it show 2 instances of the `demo-micro-service` application. You can check variety of information about these instances.
Now let's scale `demo-micro-service` to 3 instances and observe Spring Boot Admin figuring out that a new instance appeared (using Eureka Service Registry).
```bash
ip=$(kubectl get svc -o=json | jq -r '.items[] | select(.metadata.name == "admin").status.loadBalancer.ingress[0].ip'); open "http://$ip:8080"
kubectl get deployment
kubectl scale deployment demo-micro-service --replicas=3
kubectl get pod
```

### And we are done here!
