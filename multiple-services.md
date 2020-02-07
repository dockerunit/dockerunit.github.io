---
title: Testing multiple services
has_children: false
nav_order: 4
---

# Testing multiple services

This page describes how to configure more complex tests where you need more than one service and, potentially,
multiple instances of a single service.

You can link multiple descriptors to a test or test class by using the `@WithSvc` annotation. 
For each descriptor, Dockerunit creates a `Service`.
Each service by default has one replica, however, you can declare multiple replicas. For each replica, Dockerunit creates a `ServiceInstance`.

Dockerunit implements service discovery using [Consul](https://consul.io). 
It is necessary to add a `@WebHealthCheck` or a `@TCPHealthCheck` annotation to each of your descriptors, otherwise service discovery will fail and your test will not be executed.
When you run a test, the following happens:
1. Descriptor classes are interpreted to create containers.
2. Each container is wrapped into a `ServiceInstance` and instances are grouped by `Service`. Finally, services are grouped into a `ServiceContext`.
3. The `ServiceContext` is passed to the discovery provider (dockerunit-consul), which registers each `ServiceInstance` into Consul and then verifies that all the expected instances are successfully registered and `healthy`.
4. The updated `ServiceContext` is made available to the test through the `DockerUnitRule` class.
5. The test is executed.
6. The `ServiceContext` is cleaned up (containers are de-registered from Consul and deleted).

## Declaring and using multiple descriptors
Let's see a concrete example where we have two services:
- A MySQL database.
- A Spring service that uses the MySQL database.

Here is how the MySQL descriptor would look like:
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.TCPHealthCheck;

@Svc(name="mysql-db", image="mysql:8.0.18") // Service name and mysql docker image values
@TCPHealthCheck(port=3306) // Checks that MySQL is running on 3306 during service discovery
// Mounts "src/test/resources/init.sql" inside the container so your schema is executed at startup  
@Volume(host="init.sql", container="/docker-entrypoint-initdb.d/init.sql", useClasspath=true) 
public class MySQLDescriptor {
}
```

And here is how the Spring service descriptor would look like:
```java
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.WebHealthCheck;

@Svc(name="spring-svc", image="my-spring-image:latest")
@WebHealthCheck(endpoint="/health-check", port=8080)
@PublishPorts // Maps the container port 8080 to a random port on the Docker bridge
// Makes this service use Consul as DNS, 
// so that it can find the db under the "mysql-db.service.consul" name 
@UseConsulDns 
public class SpringSvcDescriptor {
}
```

Before we look at how to use these descriptors, let's clarify a couple of details:
- Both the `@TCPHealthCheck` and the `@WebHealthCheck` always refer to the container port, regardless of how this is being mapped. The actual health check is executed by Consul which runs on the same network as the containers that are created by Dockerunit. Consul is aware about the IP address of each container.
- We are not mapping the MySQL port using `@PublishPort` because our test is not going to directly talk to it.
- We are placing a `@UseConsulDns` annotation only on the `SpringSvcDescriptor` because this service is talking to MySQL and not vice versa.
- We are mapping the `spring-svc` port 8080 to a random one because we want to create two replicas. If we were to map that port to a specific one, then we would have faced a port conflict and our test would have failed. It is worth noticing that this is not a problem if we don't need to reach our service directly from the test (for instance, it is an internal service or we are running a proxy line nginx in front of our services). In that case, we can simply omit port mapping, as the clients of such service will be able to reach it using its Consul name (Consul performs DNS load balancing).

Finally, let's see how we can combine these descriptors into a single test class.

```java
import org.junit.*;
import com.github.dockerunit.core.*;
import com.github.dockerunit.core.annotation.WithSvc;
import com.github.dockerunit.junit4.DockerUnitRule;
import io.restassured.RestAssured;

// Use higher priority for MySQL, so that it runs before any service tries to connect
@WithSvc(svc=MySQLDescriptor.class, priority=10)
// Use lower priority and spin two replicas
@WithSvc(svc=SpringSvcDescriptor.class, priority=1, replicas=2)
public class MyIntegrationTest {

  @Rule
  public DockerUnitRule rule = new DockerUnitRule();
  private ServiceContext context;

  @Before
  public void setup() {
    context = DockerUnitRule.getDefaultServiceContext();
  }
  @Test
  public void testSpringSvc() {
    Service s = context.getService("spring-svc");
    ServiceInstance si = s.getInstances().stream().findAny().get();

    RestAssured
        .given()
            .baseUri("http://" + si.getIp() + ":" + si.getPort())
        .when()
            ....
  }
}
```

Need more to get started? Try looking at a [full example](https://github.com/dockerunit/examples/tree/master/spring-boot-example).