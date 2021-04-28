---
layout: post
title: JVM Monitoring in Docker
---

JVM monitoring has been one of my banes. This is the "easiest" way I've found to do it:

# Requirements

* Docker
* JVM
* Docker image running Java (somehow)

## Dockerfile

```dockerfile
FROM openjdk:8u181-jre

...
COPY entrypoint.sh .
ENTRYPOINT ["./entrypoint.sh]
```

## Shell Script

```shell
#!/bin/sh
...
java \
	-Dcom.sun.management.jmxremote.rmi.port=9090 \
	-Dcom.sun.management.jmxremote=true \
	-Dcom.sum.management.jmxremote.port=9090 \
	-Dcom.sun.management.jmxremote.ssl=false \
	-Dcom.sun.management.jmxremote.authenticate=false \
	-Dcom.sun.management.jmxremote.local.only=false \
	-Djava.rmi.server.hostname=0.0.0.0 \
	-jar application.jar
```

