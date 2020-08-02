---
layout: post
title: Introduction to Docker for Developers
tags: docker
---

A couple of years ago, I heard about Docker. I was working on a project that was in it's infancy, and I was directed at the time to look into using it, since it might be useful. Sadly, we were using cloud servers and were only allows to use RHEL 6, [which didn't really work](https://access.redhat.com/solutions/1378023). Needless to say, I kept Docker in the back of my mind, and knew I was going to come back to it when possible.

Cue a few years later: I'm no longer on that project. I'm now being drafted to support another developer in fleshing out a project he's been working on. Goal is to take his microarchitecture tool and deploy it. I had previously written tons and tons of bash scripting to accomplish this purpose, but recognized that containerization was probably the way to go. 