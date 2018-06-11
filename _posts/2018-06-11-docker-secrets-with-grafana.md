---
layout: post
title: "Docker Secrets in Grafana"
---

As my [PR](https://github.com/grafana/grafana-docker/pull/166) has now been
merged, the latest official Grafana Docker
[image](https://hub.docker.com/r/grafana/grafana/) (still in beta) has now
support for Docker secrets.  Using this version, you should be able to write
stack files similar to the following (shortened) one, provided you have the
password for the main administation user stored in the file at
`config/grafana/admin.pwd`.

```yaml
version: '3.3'

services:
  grafana:
    image: grafana/grafana:5.2.0-beta1
    environment:
      - GF_SECURITY_ADMIN_PASSWORD_FILE=/run/secrets/admin.pwd
    deploy:
      restart_policy:
        delay: 10s
        max_attempts: 10
        window: 60s
      replicas: 1
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "10"
    healthcheck:
      test: curl --fail http://localhost:3000/ || exit 1
      interval: 1m
      timeout: 10s
      retries: 3
    secrets:
      -
        source: admin-passwd
        target: /run/secrets/admin.pwd
        mode: 0444

secrets:
  admin-passwd:
    file: config/grafana/admin.pwd
```

For any environment variable that starts with `GF_` and ends with `_FILE`, the
Grafana Docker image will read the content of the file that it points at and
arrange for the environment variable with the same name but without the trailing
`_FILE` to be set before the main grafana process is started.  Using a trailing
`_FILE` is in line with other official images such as
[postgres](https://hub.docker.com/_/postgres/) or
[wordpress](https://hub.docker.com/_/wordpress/) for example.