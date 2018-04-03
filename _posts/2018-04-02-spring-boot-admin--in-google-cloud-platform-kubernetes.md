---
layout: post
title:  "Spring Boot Admin in Google Cloud Platform Kubernetes"
published: false
---

- Prerequisites
- Approaches to use Spring Boot Admin
- Preparing K8s cluster with Eureka
- Preparing Spring Boot Admin
- Deploying demo app

# Prerequisites
We need a Kubernetes cluster, Docker to build our images, Docker repository to push you custom container images to, and to pull from when you deploy.

For Kubernetes cluster and Docker repository we are going to use Google Cloud Platform aka GCP.

```bash
#you do need `brew`, if you don't have it yet https://brew.sh
brew tap caskroom/versions
brew cask install java8
brew install kubectl jq
brew cask install docker
open /Applications/Docker.app
brew cask install google-cloud-sdk
gcloud auth login
```

You will be taken to your web browser to login to your google account and allow it to use GCP.


```bash
#requirements JDK, node, docker, Google Cloud SDK, gcloud, kubectl

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
