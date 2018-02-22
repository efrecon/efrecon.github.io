---
layout: post
title: "Secured Reverse Proxying of Containers"
---

## Caddy

[Caddy](https://caddyserver.com/) is a fantastic piece of software that is both
a modern and secure web server and also integrates free HTTPS certificates from
[Let's Encrypt](https://letsencrypt.org/). Caddy has good defaults for all
things TLS, has support for QUIC and setting it it up as a reverse proxy to a
container implies writing simple
[configuration](https://caddyserver.com/docs/caddyfile) files.

## Managing Complexity

In most projects, you will be making use of the
[proxy](https://caddyserver.com/docs/proxy) directive to provide a single
entrypoint on 443 and reroute access to various containers, implementing various
services of your architecture. As the project grows, so will the number of
containers and restart will occur. As good Caddy is, it still only takes a
snapshots of the running containers when it starts. This means that managing
dependencies between all those containers will easily start to get more complex.

[caddy-gen](https://github.com/wemake-services/caddy-gen) relieves from part of
this complexity by watching containers that are present on the host and,
provided that they have setup on a number of labels, will automatically generate
a configuration file in order to proxy those containers. The implementation is
simple and builds upon the ideas of
[nginx-proxy](https://github.com/jwilder/nginx-proxy), which does pretty much a
similar job, but on top of nginx.

I have just made a few [tweaks](https://github.com/efrecon/caddy-gen) to the
original implementation for some of the internals of
[joicecare](https://www.joicecare.se/).  These are:

* The ability to have several hostnames sharing the same container proxying,
  e.g. a container serving `myapp.com` and `example.com` at the same time.
* Building Caddy from scratch, throug bringing in techniques from
  [this](https://github.com/abiosoft/caddy-docker/blob/master/Dockerfile)
  Dockerfile. This makes it possible to integrate plugins in the Caddy
  installation, and also gets rid of the ads that Caddy inserts in the headers
  when its binary is downloaded from the release pages.
  
## Future

There are probably more features that could be added to this solution to make it
usable for a wider range of use cases:

* Ability to specify random blocks of configuration files for a given proxying
  directive, in addition to the good defaults that are taken by the template.
* Support for load-balancing.
* Quick and dirty basic authentication protection of part of the
  infrastructure/project.
  
Most of these features are already present in an internal project that is based
on top of nginx, [concocter](https://github.com/efrecon/concocter) and
[le-sidekick](https://github.com/efrecon/le-sidekick) for interaction with Let's
Encrypt.