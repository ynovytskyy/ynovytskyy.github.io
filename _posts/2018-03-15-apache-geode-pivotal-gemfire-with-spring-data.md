---
layout: post
title:  "Using Apache Geode / Pivotal Gemfire with Spring Data and Spring Boot"
date:   2018-03-15 23:40:12 -0500
---

This quick guide will focus on a couple of things:
- Standing up Apache Geode to run locally as a Docker container for development purposes and all necessary configuration to serve local Java client
- Spring Boot Java client to use Apache Geode detailing potential pitfalls (that previously occurred to the author when developing for Apache Geode / Pivotal GemFire)

## Standing up Apache Geode / GemFire

 > **Prerequisites:** Docker installed and running, being able to execute `docker` commands from command line. <https://www.docker.com/community-edition#/download>

We are going to use Apache Geode official docker image from Docker Hub: <https://hub.docker.com/r/apachegeode/geode>

```bash
docker pull apachegeode/geode:latest
```

The default `run` command of the container is running GFSH (GemFire Shell) which is a shell to interact (start, configure) with Geode. So after running the following command you will end up in GFSH command prompt and continue to configure Geode from there.

```bash
docker run -p 1099:1099 -p 7070:7070 -p 8080:8080 -p 10334:10334 -p 40404:40404 -it apachegeode/geode:latest
```

## Configuring Apache Geode / GemFire

In the GFSH console:

```bash
#since Geode/GemFire is running a container but ports are mapped to localhost, we want to advertise the services to the client as if they are running on localhost
start locator --name=geode-locator --hostname-for-clients=localhost
start server --name=geode-server --hostname-for-clients=localhost
list members

create region --name=booksRegion --type=REPLICATE
list regions
```

## Running Spring Boot based Java app

Let's clone and run Spring Boot that talks to Geode / GemFire. The app uses GemFire branded integration libraries as the Geode-based libs are not GA yet.

```bash
git clone https://github.com/ynovytskyy/spring-boot-data-gemfire-pdx-client.git
cd spring-boot-data-gemfire-pdx-client
./gradlew bootRun
```

The app should be running on `localhost:8088`. You can check <http://localhost:8088/swagger-ui.html> but let's call our app's REST interface with `curl` in a separate terminal:

```bash
#list all books; empty at the beginning
curl -s localhost:8088/book | jq
#create a random book entry
curl -X POST localhost:8088/book
#check books list again
curl -s localhost:8088/book | jq
#delete all books in cache
curl -X DELETE localhost:8088/book
```

> **Note:** I'm using `jq` to format JSON output. If you don't want to install it, remove piping through `jq` from the above commands.

## Code highlights

Important places in the code to pay attention for:

```java
@Configuration
@EnableGemfireRepositories(basePackageClasses = BookRepository.class)
public class GemfireConfig {

    @Value("${org.yny.sample.gemfire.locator.host:localhost}")
    private String locatorHost;

    @Value("${org.yny.sample.gemfire.locator.port:10334}")
    private Integer locatorPort;

    @Value("${org.yny.sample.gemfire.dataModelPackage:org.yny.sample.model.*}")
    private String dataModelPackage;

    @Bean
    public ClientCacheFactoryBean gemfireCache() {
        ClientCacheFactoryBean clientCacheFactoryBean = new ClientCacheFactoryBean();
        //NB locator has to be configured on the **client** cache factory bean
        clientCacheFactoryBean.addLocators(new ConnectionEndpoint(locatorHost, locatorPort));
        //NB PDX serializer has to be configured with the data model package
        clientCacheFactoryBean.setPdxSerializer(new ReflectionBasedAutoSerializer(dataModelPackage));
        clientCacheFactoryBean.setClose(true);
        return clientCacheFactoryBean;
    }

    @Bean("booksRegion") //NB needs to match region name
    public ClientRegionFactoryBean<Object, Object> booksRegion(GemFireCache gemfireCache) {
        ClientRegionFactoryBean<Object, Object> regionFactoryBean = new ClientRegionFactoryBean<>();
        regionFactoryBean.setCache(gemfireCache);
        regionFactoryBean.setClose(false);
        regionFactoryBean.setShortcut(ClientRegionShortcut.PROXY); //NB running as a client, no local caching
        return regionFactoryBean;
    }
}
```
