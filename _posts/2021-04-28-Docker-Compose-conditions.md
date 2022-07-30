---
layout: post
title: Docker Compose and condition
tags: docker
---

As referenced in ["Docker Compose" and `depends_on`](../_posts/2021-04-27-Docker-Compose-and-depends_on.md), I've recently learned that the `depends_on` configuration option supports the `condition` in Version 3. For me, this is a big deal -- I spend a lot of time working on Docker Compose systems for local development and for deployments. One of the biggest issues that I've run into is trying to correctly time containers.

The goal of using `condition` is to allow Docker Compose to correctly start dependant services based on a certain state. The definition of the `docker-compose.yml` file can be found [here](https://github.com/compose-spec/compose-spec/blob/master/schema/compose-spec.json), with the `condition` definition looking like this:

```json
"condition": {
  "type": "string",
  "enum": ["service_started", "service_healthy", "service_completed_successfully"]
}
```

The `condition` configuration has 3 options:
* `service_started`: The default. If the `condition` configuration is not applied, this is used. The dependant starts after the required service has started. Nothing else is relevant.
* `service_healthy`: The dependant starts after the required service has reached a `healthy` state, managed by the `HEALTHCHECK` configuration. This is particularly helpful for starting things like applications after the database has reached a healthy state.
* `service_completed_successfully`: The dependant starts after the required service has exited with an exit code of 0. This is useful if you need to perform certain one-shot tasks in series, like database migrations, or initialization.

Let's see some examples.

# Using `service_started`
```
# docker-compose.yml
version: "3"
services:
  nginx:
    image: nginx
	ports:
	  - "80:80"
	depends_on:
	  app:
	    condition: "service_started"
  app:
    image: node
```

This is a good example of a standard reverse-proxy situtation. The `nginx` container depends on the application container, and like uses the `proxy_pass` directive to redirect some requests to the app.

Based on my experience, nginx requires that the upstream is available before the nginx container can correctly start. In the case of the `app` service, the startup time might be relatively quick, so the `nginx` service can simply wait for it to have been started, but nothing else is required.

One thing to note is that because `service_started` is the default value, this file can also be written as such:

```
# docker-compose.yml
version: "3"
services:
  nginx:
    image: nginx
	ports:
	  - "80:80"
	depends_on:
	  - app
  app:
    image: node
```

# Using `service_healthy`
```
# docker-compose.yml
version: "3"
services:
  nginx:
    image: nginx
	ports:
	  - "80:80"
	depends_on:
	  app:
	    condition: "service_healthy"
  app:
    image: node
	healthcheck: curl --fail -I http://localhost:8080/status || exit 1
```

In this version of the Docker Compose file, the `app` image now has a healthcheck. This curl command checks to see if the URL of `localhost:8080/status` is returning a good value. If it isn't it returns a 1. While the container is starting, the status will be in `starting`. Once the healthcheck succeeds, it is moved to `healthy`. Only at that point does the `nginx` container start up.

This sort of thing can be used to reasonably time the startup of certain containers in the `docker-compose.yml`. Users don't need to resort to using bash scripts to time the startup, they can encode the dependency and the requirements right in to the healthchecks and `depends_on`.

One problem with this condition is that you need to make sure your required service has a healthcheck of some sort, and it's viable for the service to actually reach that state. This means that if the base image you are working with doesn't come with a healthcheck, you will need to write your own. This itself isn't normally hard, but if you are waiting for a certain state of the application, you might need to try a number of things to get to that state.

Alternatively, if the dependant service has `restart: always` or `restart: on-failure` set and the dependant application exits or fails when the requred service is not in a good state, nothing of value has been lost. The `restart` configurations will restart the dependant service (based on the configuration), allowing the service to effectively wait until all of it's requirements are met.

# Using `service_completed_successfully`
```
# docker-compose.yml
version: "3"
services:
  nginx:
    image: nginx
	ports:
	  - "80:80"
	depends_on:
	  app:
	    condition: "service_healthy"
  app:
    image: node
	healthcheck: curl --fail -I http://localhost:8080/status || exit 1
	depends_on:
	  flyway:
	    condition: "service_completed_successfully"
  postgres:
    image: postgres
  flyway:
    image: flyway
	depends_on:
	  postgres:
	    condition: "service_healthy"
```

In this scenario, we have a [Flyway](https://flywaydb.org/) for database migrations. This container is dependant on the [PostgreSQL](https://www.postgresql.org/) database container. We want to make sure that the postgres container is healthy before the database migration is completed.

The `app` service is not waiting for the `flyway` service to be healthy, but rather complete. Since the `flyway` service will execute fully, and is not a long running process, the `app` is waiting for `flyway` to successfully complete. If `flyway` for some reason exits with a non-zero status code, `app` will never start, which means in turn `nginx` will never start.

Much list `service_healthy`, if the state of the required service is unknown, failing the status and using `restart` configurations can achieve much the same result, often without needing to invest into setting up healthchecks (though you should) and configuring the Docker Compose file.

---

While the value of the `condition` configuration is something I'm going to be testing in the short term, in the longer term it's likely a key piece of the Docker Compose spec that you should not only be aware of, but be willing to try.