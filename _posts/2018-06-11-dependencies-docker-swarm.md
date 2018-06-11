---
layout: post
title: "Solving Dependencies in Docker Swarm"
---

When/If moving from Docker Compose files to Stack files in Docker Swarm, you
might have problems solving dependencies and especially starting order as
[depends_on](https://docs.docker.com/compose/compose-file/#depends_on) is not
supported when
[deploying](https://docs.docker.com/engine/reference/commandline/stack_deploy/)
a stack in swarm mode.  One solution is to arrange for your services to
implement a network server and wait for this port to be opened and respond
before going on with running a given service.

Let's run roughly through the example of running [grafana](https://grafana.com/)
in a scalable way, for example using Redis and Postgres for, respectively,
runtime and persistent data.  To make this more complex, let's suppose that the
TOML configuration file of grafana is the result of another service.  A regular
Grafana Docker installation neither has an entrypoing, nor a run command as it
is mostly [configured](http://docs.grafana.org/installation/docker/) using a
combination of environment variables and the TOML configuration file.  To
arrange for the other services to be ready, you could have the following
(snippet) entrypoint:

```yaml
    entrypoint: >-
      wait-for.sh pg_grafana:5432 -t 120 -v --
        wait-for.sh redis:6379 -t 120 -v --
          wait-for.sh grafana-init:8080 -t 120 -v --
            /run.sh
```

In that example, `pg_grafana`, `redis` and `grafana-init` are the name of the
services that implement (respectively) the postgres database, Redis and the
initialisation of the TOML configuration. Creating them is (almost) out of the
scope of this post... The implementation for
[wait-for.sh](https://gist.github.com/efrecon/86456960e2110b287632fd7f42c1cd31)
is available as a gist. It deviates slightly from the
[original](https://github.com/Eficode/wait-for) as it prefers `nc` for trying to
establish connection to remote servers, but is also able to use pure bash
[constructs](https://www.tldp.org/LDP/abs/html/devref1.html) whenever `bash` is
available.

`grafana-init` is a bit special as it typically generates a configuration file
depending on a number of parameters.  In a regular installation, such a service
would "die" once it had created the configuration file. However, instead it can
wait forever once it has performed its initialisation job. I have experimented
with relaying another external service with `socat`, as this is available in
most distributions. I typically call the TOML generator with a commane-line
option similar to `-r 8080:icanhazip.com:80`, you will notice that `8080` is the
port that the grafana service was waiting for. The implementation is as follows,
provided the content of the `-r` switch is contained in the variable `RELAY`.
Even if the remote service was not available, initialisation would still work as
`socat` would still respond on `8080` on incoming client requests.

```shell
if [ -n "$RELAY" ]; then
    lport=$(echo "${RELAY}"|cut -d: -f1)
    remote=$(echo "${RELAY}"|cut -d: -f2)
    rport=$(echo "${RELAY}"|cut -d: -f3)
    socat TCP4-LISTEN:${lport},su=nobody,fork,reuseaddr TCP4:${remote}:${rport}
fi
```