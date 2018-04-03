---
layout: post
title:  "Spring Boot Admin in Kubernetes"
published: false
---

- Prerequisites
  - Tools
  - Set up K8s cluster on GCP
- Approaches to use Spring Boot Admin
- Spring Boot Admin with Eureka
  - Setup Eureka Service Registry
  - Setup Spring Boot Admin
  - Deploying demo micro-service

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

## Setup Spring Boot Admin

```
#requirements JDK, node, docker, Google Cloud SDK, gcloud, kubectl
docker build --no-cache -f src/main/docker/Dockerfile -t sba-server-eureka .

#build Spring Boot Admin service
git clone git@github.com:codecentric/spring-boot-admin.git
cd spring-boot-admin
mvnw install

#gcloud projects list - to find out project id
gcloud beta auth configure-docker
#?? gcloud components install docker-credential-gcr

cd spring-boot-admin-samples/spring-boot-admin-sample-eureka
docker build --no-cache -f src/main/docker/Dockerfile -t sba-server-eureka .
cd ../..
docker tag sba-server-eureka gcr.io/cso-ynovytskyy/sba-server-eureka

#doc says: docker push gcr.io/cso-ynovytskyy/sba-server-eureka
gcloud docker -- push gcr.io/cso-ynovytskyy/sba-server-eureka

kubectl scale deployment micro-service-deployment --replicas=3

```
