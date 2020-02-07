---
title: Basics
has_children: false
nav_order: 2
---

# Basics

This page covers most of the common configs one might want to set on a Docker container.
Please note that `@Svc` and `@WebHealthCheck` (or `@TCPHealthCheck`) are mandatory for each descriptor, so they will be present in each example.

### Mounting a volume
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.*;

@Svc(name="my-service", image="my-docker-image")
@WebHealthCheck(port=8080)
@Volume(host="/home/myuser/application.properties", container="/application.properties")
public class MyDescriptor {}
``` 
Useful but not quite enough for a test. Dockerunit supports mounting of resources from the test classpath.

```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.*;

@Svc(name="my-service", image="my-docker-image")
@WebHealthCheck(port=8080)
@Volume(host="application.properties", container="/application.properties", useClasspath=true)
public class MyDescriptor {}
``` 
The snippet above mounts the `application.properties` file from the `src/test/resources` directory inside your project.

This way you can easily mount different config files for different tests inside your Docker container.

### Passing an environment variable
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.*;

@Svc(name="my-service", image="my-docker-image")
@WebHealthCheck(port=8080)
@Env({"FOO=foo", "BAR=bar"})
public class MyDescriptor {}
``` 
This is equivalent to `docker run -e FOO=foo -e BAR=bar my-docker-image`

### Exposing a container port to the host
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.*;

@Svc(name="my-service", image="my-docker-image")
@WebHealthCheck(port=8080)
@PublishPort(host=9080, container=8080) 
public class MyDescriptor {}
``` 
This is equivalent to `docker run -p 9080:8080 my-docker-image`

### Exposing a container port to a random host port (to avoid port conflicts)
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.*;

@Svc(name="my-service", image="my-docker-image")
@WebHealthCheck(port=8080)
@PublishPorts
public class MyDescriptor {}
``` 
This is equivalent to `docker run -P my-docker-image`

### Provide or override the command to execute once the container starts
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.*;

@Svc(name="my-service", image="my-docker-image")
@WebHealthCheck(port=8080)
@Command({"cat", "/etc/hosts"})
public class MyDescriptor {}
``` 
This is equivalent to `docker run my-docker-image cat /etc/hosts`

Need more to get started? Try looking at a [full example](https://github.com/dockerunit/examples/tree/master/spring-boot-example).