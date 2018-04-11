---
layout: post
title: "concocter v1.0"
---

[concocter](https://github.com/efrecon/concocter) has just reached an official
[v1.0](https://github.com/efrecon/concocter/releases/tag/1.0) release.
`concocter` is my own take on the init process in containers. It offers features
already found in other solutions such as [supervisord](http://supervisord.org/)
or [docker-gen](https://github.com/jwilder/docker-gen) with enough twists for
justifying the effort of writing yet another tool in a similar vein.

## Rationale

The rationale of `concocter` is to acquire variables from a number of remote of
local sources, to dynamically generate (configuration) files with the content of
these variables and to launch one or several processes once the files have been
generated. `concocter` can be placed in the background to continuously perform
these tasks, thus being able to regenerate the file as soon as a variable has
changed. In that case, `concocter` will restart the process, or request it to
reload its configuration using regular signals.

`concocter` has support for a range of sources for these variables. This
includes information about other containers running on the same Docker host, but
also the content of files (good for integration with Docker secrets), or the
content of a external HTTP(S) resources. The latter faciliates the use of, for
example, an internal key-value store with an RESTish interface for the storage
and access of cluster or project wide configuration variables. Using `concocter`
together with a capable reverse-proxy server such as
[nginx](https://www.nginx.com/) or [caddy](https://caddyserver.com/) provides
ways to automatically proxy containers carrying "instructions" as Docker labels
or environment variables.

## Discussion and Community

The [github](https://github.com/efrecon/concocter/issues) project provides
support for issue tracking and enhancement proposals.