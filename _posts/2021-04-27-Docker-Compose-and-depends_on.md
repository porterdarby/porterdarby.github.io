---
layout: post
title: Docker Compose and `depends_on` -- fixing documentation
tags: docker
---

One of the more interesting problems I ran into recently was the use of `depend_on` in Docker Compose files.

I was working on setting up [Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/index.html), and I noticed there was a [Docker Compose](https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml) file that would stand up Airflow and all of it's dependencies. Everything looked good -- save for [these five lines at the top](https://github.com/apache/airflow/blob/master/docs/apache-airflow/start/docker-compose.yaml#L58-L62):

```
  depends_on:
    redis:
      condition: service_healthy
    postgres:
      condition: service_healthy
```

The first time seeing this, I was confused because the [Docker Docs](https://docs.docker.com/compose/compose-file/compose-file-v3/#depends_on) says that "Version 3 no longer supports the `condition` form of `depends_on`."

![[Version 3 no longer supports the condition form of depends_on.png]]

I did the logical thing and made [a PR for Airflow](https://github.com/apache/airflow/pull/15509) that would remove those lines. They weren't doing anything, right?

Wrong.

As [mik-laj points out](https://github.com/apache/airflow/pull/15509#issuecomment-826120396), Docker Compose actually does support `condition` in the Version 3 form of a `docker-compose.yml`, the documentation is just wrong.

[Testing it out](https://github.com/porterdarby/docker-compose-condition-test), I found out that mik-laj was right. I wasn't expecting them to be wrong, but I needed to confirm. This was a documentation problem on a tool that I use... daily. If I can use `condition`, it'll help me make my compose files even better.

Well, if the documentation is wrong, I gotta fix it. [PR](https://github.com/docker/docker.github.io/pull/12746) was pushed to  the `docker.github.io` repo, and was eventually approved and merged. I imagine it'll be updated in the next couple of weeks on the side.

---

While the story itself is rather boring, it broght me an immense sense of satisfaction to not only find out that something that is documented is wrong, but be able to do something about it. I was able to put my fingers to the keyboard.
