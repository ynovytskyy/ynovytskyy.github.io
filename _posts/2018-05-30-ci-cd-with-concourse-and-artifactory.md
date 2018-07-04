---
layout: post
title:  "CI/CD with Concourse and Artifactory"
published: false
---

https://www.jfrog.com/confluence/display/RTF/Installing+with+Docker

```
docker pull docker.bintray.io/jfrog/artifactory-oss
docker run --name artifactory -d -p 8081:8081 docker.bintray.io/jfrog/artifactory-oss:latest
docker run --name artifactory-pro -d -v /var/opt/jfrog/artifactory:/var/opt/jfrog/artifactory -p 8081:8081 docker.bintray.io/jfrog/artifactory-pro:latest
docker volume create --name artifactory5_data
docker run --name artifactory-pro -d -v artifactory5_data:/var/opt/jfrog/artifactory -p 8081:8081 docker.bintray.io/jfrog/artifactory-pro:latest
```
