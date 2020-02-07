
# Home
{::nomarkdown}<script async defer src="https://buttons.github.io/buttons.js"></script>{:/}



[![Discord](https://img.shields.io/discord/587583543081959435.svg?style=flat)](https://discordapp.com/channels/587583543081959435/587583543081959437)
&nbsp;
[![License](https://img.shields.io/github/license/dockerunit/dockerunit-core.svg?style=flat)](https://choosealicense.com/licenses/apache-2.0/)
&nbsp;
{::nomarkdown}<a class="github-button" href="https://github.com/dockerunit/dockerunit-core/stargazers" data-icon="octicon-star"  aria-label="Star dockerunit/dockerunit-core on GitHub">us on Github</a>{:/}

Dockerunit is an extensible framework for testing of dockerised services and
applications.
It enables linking of Docker images to Java tests by
means of Java annotations.
It works with both JUnit 4 and 5, however support for other testing frameworks can be easily added.
You can think of Dockerunit as a docker-compose for Java tests.

Dockerunit leverages useful tools like
[docker-java](https://github.com/docker-java/docker-java) and
[Consul](https://www.consul.io/) to provide the
following main features:
1. Automatic pull of images referencing a registry.
2. Service discovery based on Consul (alternative discovery
providers can be plugged in).
3. Container port mapping.
4. Volume mapping allowing relative paths from test classpath, so you can
easily mount test config files.
5. Support for multiple instances of a service, so you can check how the tested services would work on environments like [Kubernetes](https://kubernetes.io/).
6. A simple mechanism similar to the
[Java Validation Framework](https://jcp.org/en/jsr/detail?id=303) for you to
add custom annotations.

## Usage
You can enable Dockerunit by adding the following dependencies to you POM file
(set `dockerunit.version` property to the version you intend to use).

[Core](https://github.com/dockerunit/dockerunit-core): builds containers using Java-based descriptors

[![CircleCI](https://img.shields.io/circleci/build/gh/dockerunit/dockerunit-core/master.svg?style=flat)](https://circleci.com/gh/dockerunit/dockerunit-core/tree/master)
&nbsp;
[![Maven](https://img.shields.io/maven-central/v/com.github.dockerunit/dockerunit-core.svg?style=flat)](https://search.maven.org/search?q=g:com.github.dockerunit%20AND%20a:dockerunit-core&core=gav)
&nbsp;
[![Javadoc](https://javadoc.io/badge/com.github.dockerunit/dockerunit-core.svg)](https://www.javadoc.io/doc/com.github.dockerunit/dockerunit-core)
&nbsp;
```xml
<dependency>
  <groupId>com.github.dockerunit</groupId>
  <artifactId>dockerunit-core</artifactId>
  <version>${dockerunit.version}</version>
  <scope>test</scope>
</dependency>
```
[Consul](https://github.com/dockerunit/dockerunit-consul): ensures service discovery and health-checking

[![CircleCI](https://img.shields.io/circleci/build/gh/dockerunit/dockerunit-consul/master.svg?style=flat)](https://circleci.com/gh/dockerunit/dockerunit-consul/tree/master)
&nbsp;
[![Maven](https://img.shields.io/maven-central/v/com.github.dockerunit/dockerunit-consul.svg?style=flat)](https://search.maven.org/search?q=g:com.github.dockerunit%20AND%20a:dockerunit-consul&core=gav)
&nbsp;
[![Javadoc](https://javadoc.io/badge/com.github.dockerunit/dockerunit-consul.svg)](https://www.javadoc.io/doc/com.github.dockerunit/dockerunit-consul)
&nbsp;
```xml
<dependency>
  <groupId>com.github.dockerunit</groupId>
  <artifactId>dockerunit-consul</artifactId>
  <version>${dockerunit.version}</version>
  <scope>test</scope>
</dependency>
```
[JUnit4](https://github.com/dockerunit/dockerunit-junit4): connects service discovery and container clean-up to the JUnit test life cycle

[![CircleCI](https://img.shields.io/circleci/build/gh/dockerunit/dockerunit-junit4/master.svg?style=flat)](https://circleci.com/gh/dockerunit/dockerunit-junit4/tree/master)
&nbsp;
[![Maven](https://img.shields.io/maven-central/v/com.github.dockerunit/dockerunit-junit4.svg?style=flat)](https://search.maven.org/search?q=g:com.github.dockerunit%20AND%20a:dockerunit-junit4&core=gav)
&nbsp;
[![Javadoc](https://javadoc.io/badge/com.github.dockerunit/dockerunit-junit4.svg)](https://www.javadoc.io/doc/com.github.dockerunit/dockerunit-junit4)
&nbsp;
```xml
<dependency>
  <groupId>com.github.dockerunit</groupId>
  <artifactId>dockerunit-junit4</artifactId>
  <version>${dockerunit.version}</version>
  <scope>test</scope>
</dependency>
```

## How it works
Building tests with Dockerunit consists of two main steps:
1. Defining one or more service descriptors.
2. Using the descriptors from test classes.

### 1. Defining a service descriptor
A service descriptor is a class that instructs Dockerunit about how to create
Docker containers, given a Docker image that you have previously created.
Here is a simple descriptor for a Spring service that is listening on port 8080.

```java
import com.github.dockerunit.core.annotation.PublishPort;
import com.github.dockerunit.core.annotation.Svc;
import com.github.dockerunit.discovery.consul.annotation.WebHealthCheck;


// Gives a name to the service and selects the docker image to use. Consul will put a dns entry on `my-spring-service.service.consul`.
@Svc(name="my-spring-service", image="my-spring-service-image:latest")

/* Maps the container port 8080 on the same host port number
(equivalent to `docker run -p 8080:8080 my-spring-service-image:latest`) */
@PublishPort(container = 8080, host = 8080)


/* Tells Consul how to monitor the state of the service.
You must always provide a health check endpoint.
If not, Dockerunit cannot detect whether the service has started successfully.
'port' represents the port that is exposed by the container. */
@WebHealthCheck(port=8080, endpoint="/health-check")
public class MyServiceDescriptor {
}
```

### 2. Using the service from a test class

The best way to activate Dockerunit is by declaring a rule.
Using a rule allows you to perform service startup/discovery either once per
test class execution (`@ClassRule`) or before each test (`@Rule`).

Moreover, you can use `@ClassRule` in conjunction with JUnit `@SuiteClasses`, which allows you to perform service startup/discovery once and then execute several test classes
rapidly.


It's now time to write an actual test.
The following example uses RestAssured, but you can choose any library to hit
the service endpoints.
We are testing that our service starts correctly and that the health-check responds
with a 200 status code.

```java
import org.junit.Before;
import org.junit.Test;
import org.junit.Rule;

import com.github.dockerunit.core.Service;
import com.github.dockerunit.core.ServiceContext;
import com.github.dockerunit.core.ServiceInstance;
import com.github.dockerunit.core.annotation.WithSvc;
import com.github.dockerunit.junit4.DockerUnitRule;
import io.restassured.RestAssured;

@WithSvc(svc=MyServiceDescriptor.class) // Selects the previously defined descriptor
public class MyServiceTest {

  @Rule
  public DockerUnitRule rule = new DockerUnitRule();
  private ServiceContext context;

  @Before
  public void setup() {
    context = DockerUnitRule.getDefaultServiceContext();
  }
  @Test
  public void healthCheckShouldReturn200() {
    // Gets the service based on value in the @Svc annotation
    Service s = context.getService("my-spring-service");
    // Selects an available instance (you could declare more than one)
    ServiceInstance si = s.getInstances().stream().findAny().get();

    /* Uses the IP and port of the instance.
        The port could be dynamic if @PublishPorts is used */
    RestAssured
        .given()
            .baseUri("http://" + si.getIp() + ":" + si.getPort())
        .when()
            .get("/health-check") // Hits the health-check endpoint
        .then()
            .assertThat()
            .statusCode(200);
  }
}
```

This is Dockerunit in a nutshell.
1. It uses Java-class-based descriptors to instantiate one or more Docker containers.
2. It makes sure that each of them started successfully and that the discovery
provider (for now Consul) can monitor their state.
3. It provides you with a ServiceContext instance that you can use to select the
services and endpoints to hit.
4. It cleans up the containers after the the test execution (also when the
test fails unexpectedly).
