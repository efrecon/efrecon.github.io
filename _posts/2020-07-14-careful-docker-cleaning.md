---
layout: post
title: "Careful Docker Cleaning"
---

As time goes by, Docker will leave "remains" behind and it is good practice to
clean away old cruft from time to time. While this happens mostly of dev
machines, as goals and requirements shift quickly, it also happens on production
servers. Unused old images will be left behind, perhaps dynamic or named volumes
issuing from a debugging session, etc. The documented solution to this is to run
[`docker system prune`][prune] from time to time.

  [prune]: https://docs.docker.com/engine/reference/commandline/system_prune/

Wether you are operating [cattle or pets][cattle-pets], you will probably want
to automate Docker resource cleanup, and using [prune] is a point of no return,
meaning it could even lead to possible data loss when pruning volumes. The
open-source [docker-prune] project tries to provide an alterative that is more
conservative in the pruning decisions that it takes. For recurring operations, I
recommend [dockron].

  [cattle-pets]: http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/
  [docker-prune]: https://github.com/YanziNetworks/docker-prune
  [dockron]: https://github.com/efrecon/dockron

[docker-prune] is a POSIX shell script that will prune exited containers,
dangling volumes and dangling images with the following twist:

## Containers

All exited, dead and *stale* containers will be removed, and this provides
filtering capabilities similar to the [prune][cprune] command. Containers that
have a name that was automatically generated by Docker at creation time are
automatically selected. In addition, when removing containers, the script can
consider only a subset of the containers.

Exited and dead containers are as reported by Docker. Stale containers are
containers that are created but have not moved to any other state after a given
timeout.

In addition, it is possible to forcedly remove ancient, but still running
containers. This might be a dangerous operation, and it is turned off by
default.

  [cprune]: https://docs.docker.com/engine/reference/commandline/container_prune/

## Images

All dangling and orphan images will be removed. This also provides filtering
capabilities similar to the [prune][iprune] command. When removing images, the
script will only consider images that were created a long time ago (6 months by
default).

Dangling images are layers that have no relationship to any tagged images.
Orphan images are images that are not used by any container, whichever state the
container is in (including created or exited state).

  [iprune]: https://docs.docker.com/engine/reference/commandline/image_prune/

## Volumes

All "empty" dangling volumes will be removed. The script will count the files
inside the volumes, only removing the ones which have less than an optional
number of files, which defaults to `0`. In addition, the script will is able to
focus on subsets of the dangling volumes. Volumes that have a name that was
automatically generated are automatically selected. File count is achieved
through mounting the volumes into a temporary [busybox] container.

  [busybox]: https://hub.docker.com/_/busybox

## Fun Fact

The script dynamically parses the official go [implementation] to detect
containers which names were automatically generated. Out of this implementation,
[this][wozniak] is probably the best line of code ever written!

  [go]: https://golang.org/
  [implementation]: https://github.com/moby/moby/blob/master/pkg/namesgenerator/names-generator.go
  [wozniak]: https://github.com/moby/moby/blob/3f3676484459a9f5ec287f09735cc018a74f3cc5/pkg/namesgenerator/names-generator.go#L844