---
layout: post
title: "Docker API in Tcl"
---

The main goal of my Docker API
[implementation](https://github.com/efrecon/docker-client) in
[Tcl](https://www.tcl.tk/) is to cover most of the official
[API](https://docs.docker.com/reference/api/docker_remote_api/) while providing
a programming interface that feels Tcl-ish. To that end, it builds upon the
Tk-syle of programming that creates a context object and then creates a command
with the same name as the object to perform most further operations.

The implementation was lagging behind and a
[recent](https://github.com/efrecon/docker-client/commit/1bbf418258006ebfaf9e081af244ef3ef139c0fd)
restructuring have started to bring it to par with the currernt state of the
Docker API itself. The restructuring matches the Docker CLI
[restructuring](https://github.com/moby/moby/pull/26025) that happened with the
[1.13.0](https://docs.docker.com/release-notes/docker-engine/#1130-2017-01-18)
version. Recent additions to the Tcl command set provide bridges to the
following sub-command trees of the API and CLI:

* [container](https://docs.docker.com/engine/reference/commandline/container/)
* [image](https://docs.docker.com/engine/reference/commandline/image/)
* [service](https://docs.docker.com/engine/reference/commandline/service/)
* [secret](https://docs.docker.com/engine/reference/commandline/secret/)
* [config](https://docs.docker.com/engine/reference/commandline/config/)
* [node](https://docs.docker.com/engine/reference/commandline/node/)
* [volume](https://docs.docker.com/engine/reference/commandline/volume/)

There still remain a number of commands, but these new implementations provide a
[consistent](https://github.com/efrecon/docker-client#api-principles) API that
is hopefully easier to work with than the previous uncategorised API, itself
loosely modelled after the previous CLI model. Work with the integration of
these new API calls is driven by the necessity to integrate some of these calls
in [dockron](https://github.com/efrecon/dockron) so as to be able to perform
regular tasks on services within a Swarm, e.g. scale up/down at known times,
restart services, etc. 

(Funny fact of the day... Apparently, this API implementation made its way to
[HN](https://news.ycombinator.com/item?id=9196178) soon after I announced it the
first time!)