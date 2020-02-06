---
title: Running tests
has_children: false
nav_order: 3
---

# Running tests

Before running the tests, make sure you know the ip of the Docker bridge network interface.
This is usually `172.17.42.1`, however some distros and Docker for macOS might use a different ip.
To find out the ip of the Docker bridge interface, type the following:

```bash
docker inspect --format='{% raw %}{{range .IPAM.Config}}{{println .Gateway}}{{end}}{% endraw %}' bridge
```

If the result is not an IP address, just type the following:

```bash
docker inspect bridge | grep Gateway
```

Let's assume you have configured your project to run all tests during the `test` phase (although you might want to use the `verify` phase for integration tests, which is where you might mostly use Dockerunit).
If you are on Linux and the ip was `172.17.42.1` you can run the tests as follows:

```bash
mvn test
```

If you are on Linux but you have a different ip (for example `172.17.0.1`), run the following:

```bash
mvn test -Ddocker.bridge.ip=172.17.0.1
```

If you are on Mac, run the following:

```
mvn test -Ddocker.host=localhost -Ddocker.bridge.ip=172.17.0.1
``` 

## Windows users
Unfortunately we haven't been able to test Dockerunit on Windows yet, however, Docker support should work fairly similarly to the way it does on macOS. Please get in touch through our chat, so that we can help you getting up and running.

Need more to get started? Try looking at a [full example](https://github.com/dockerunit/examples/tree/master/spring-boot-example).

   