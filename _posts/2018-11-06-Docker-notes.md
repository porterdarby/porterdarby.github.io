---
layout: post
title: "Docker Notes"
---

One of the tools I've been using the last couple of months to great success is [Docker](https://github.com/docker/docker-ce). These days it's almost heresy if you haven't at least heard of Docker, let alone messed around with it. As a developer, I've found it to be very useful in my development processes. The [documentation](https://docs.docker.com/get-started/) is quite solid, and it's been pretty easy to figure most things out.

Here is a collection of things about docker that I've come to learn over time. It's a collection of tips, tricks, best practices, and the like.

* You cannot mount a volume during build time. If you are in need of a dynamic image, see if you can opt for a dynamic container instead.
* `Dockerfile`s can be placed most anywhere. If you are developing a Java-Maven application, there is logic to placing the Dockerfile in the same directory as the `pom.xml`, and then just referencing the output in the `target` directory.
* At one point Java and Docker didn't like to play nicely with each other. These days, that isn't something to worry about.
* You can build a "binary" Docker image by using the executable in the image as the `ENTRYPOINT` and then have the arguments passed in and override the `CMD` flags. For example:

```
ENTRYPOINT ["ls"]
CMD ["-al"]
```
  * The default functionality of this image when run will be to run `ls -al`. If you pass any arguments to the container as it runs, the `CMD` flag will be ignored in exchange for the command line arguments, e.g. `docker run --rm ls:latest -altr` will run `ls -altr` instead.

* Utilizing the `.dockerignore` file is helpful in making sure you aren't passing a large amount of data to the daemon. I personally use a whitelisting system to reduce the amount of source files and the like that are added unnecessarily.
* Multi-stage builds allow you to put the entire build process into a Docker image for consistent replication. The downside is that you might need to manually set up the multistage process via tag references and bash files.

**Last updated 2018-11-06.**
