---
title: Customizing Dockerunit
has_children: false
nav_order: 5
---

# Customizing Dockerunit

Every time you start using a new tool, especially if you are enjoying it, it's just a matter of time before
you hit the wall of a missing feature.
For this reason, Dockerunit has been built with extensibility in mind!

Dockerunit allows you to customize container creation in two ways:
- By using the `@ContainerBuilder` pass through annotation.
- By defining your own annotations.

## Using @ContainerBuilder

Behind the scenes, Dockerunit creates Docker containers using the `CreateContainerCmd` interface from [docker-java](https://github.com/docker-java/docker-java).
You can get access to an instance of `CreateContainerCmd`, before the command is sent to Docker, by using the `@ContainerBuilder` annotation. Let's see how to.

Imagine you want to set a specific DNS server for your service.
Here is how your descriptor would look like if you want to achieve that using `@ContainerBuilder`:

```java
import com.github.dockerjava.api.command.CreateContainerCmd;
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.WebHealthCheck;

@Svc(name="my-svc", image="my-image:latest")
@WebHealthCheck(port=8080)
@PublishPort(container=8080, host=8080)
public class MyDescriptor {

    @ContainerBuilder
    public CreateContainerCmd build(CreateContainerCmd cmd) {
        return cmd.withHostConfig(cmd.getHostConfig().withDns("8.8.8.8"));
    }
}
```

For this to work, your method must both accept a parameter and return a value of type `CreateContainerCmd`.
Dockerunit makes sure that your method is called after all the other annotations have been processed, so that:
1. You are sure that your changes are not overwritten.
2. You can still overwrite any of the settings that are coming from other annotations, if needed.

Once you are happy with the effect of your `@ContainerBuilder`, you can easily make its usage declarative by evolving it into a 
custom annotations.

## Defining custom annotations
Plugging a new annotation into Dockerunit requires two simple steps:
1. Defining an extension annotation.
2. Building an extension interpreter.

Let's create a `@Dns` annotation that sets a specific DNS server for our container:

```java
import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import com.example.annotations.DnsExtensionInterpreter;

@Retention(RUNTIME)
@Target(TYPE)
@ExtensionMarker(DnsExtensionInterpreter.class)
public @interface Dns {
    String value();
}
```

The key element is the `@ExtensionMarker` annotation which wires this annotation to its interpreter.
Let's see how the interpreter looks like:
```java
import com.github.dockerjava.api.command.CreateContainerCmd;
import com.github.dockerunit.core.annotation.ExtensionInterpreter;
import com.github.dockerunit.core.internal.ServiceDescriptor;
import com.example.annotations.Dns;

public class DnsExtensionInterpreter implements ExtensionInterpreter<Dns> {

    @Override
    public CreateContainerCmd build(ServiceDescriptor sd, CreateContainerCmd cmd, Dns dns) {
        return cmd.withHostConfig(cmd.getHostConfig().withDns(dns.value()));
    }
```
Finally, we can use the newly created annotation to replace the `@ContainerBuilder` method.

```java
import com.github.dockerjava.api.command.CreateContainerCmd;
import com.github.dockerunit.core.annotation.*;
import com.github.dockerunit.discovery.consul.annotation.WebHealthCheck;
import com.example.annotations.Dns;

@Svc(name="my-svc", image="my-image:latest")
@WebHealthCheck(port=8080)
@PublishPort(container=8080, host=8080)
@Dns("8.8.8.8")
public class MyDescriptor {
}
```

Simple enough? Why not playing with a real [example](https://github.com/dockerunit/examples/tree/master/spring-boot-example)!
