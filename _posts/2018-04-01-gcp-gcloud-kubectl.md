---
layout: post
title:  "gcloud, kubectl commands cheatsheet"
published: false
---

```bash
#init and/or login
gcloud init
gcloud config set compute/zone us-east1-b
gcloud config set compute/region us-east1
gcloud auth list
gcloud auth login

gcloud compute machine-types list

#create project
gcloud projects create cso-ynovytskyy
gcloud projects list
gcloud config set project cso-ynovytskyy

#create Kubernetes cluster and configure `kubectl`
gcloud container clusters create kub-cluster
gcloud container clusters get-credentials kub-cluster
gcloud container clusters list

#create Kubernetes workloads
kubectl cluster-info
kubectl expose service sba-eureka-service --port=8080 --target-port=8080 --name=sba-lb --type=LoadBalancer

kubectl apply -f ./spring-boot-admin.yml

kubectl get pod
kubectl get svc
kubectl get rs
kubectl get deployment

kubectl delete pod --all
kubectl delete svc --all
kubectl delete deployment --all
kubectl delete rs --all
```
