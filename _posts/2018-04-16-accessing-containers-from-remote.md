---
layout: post
title: "Accessing Containers from Remote"
---

Sometimes (oftentimes?), when debugging Docker based setup, you wish you could
access a service running in a container from the outside. Typically, these
would be services that are internal to the cluster architecture, but should not
be exposed for remote access, e.g. a database, a pub/sub queue, etc. Luckily,
the only thing you really need is access to a remote SSH server and accept to
(temporarily) add an SSH client to your container. Here is how, documented for
an [alpine](https://hub.docker.com/_/alpine/) container, but steps will be the
same for containers based on other distributions.

## Exporting the Service

Start by jumping into your container using docker
[exec](https://docs.docker.com/engine/reference/commandline/exec/).

```console
$ docker exec -it <containerID> ash
```

Then from within the container, make sure to install an SSH **client**. No need
for a server here.

```console
$ apk add --no-cache openssh-client
```

Once installation has succeeded, establish a reverse tunnel onto your host.
Provided that you have a server, within your container, that listens on port
`8088`, a command similar to the following will make the same port available at
the remote host and return back to the command line of the container (this is
the meaning of `-fNT`). The command will require you to login at the remote
host, etc. Note that if you did not want to use the same port at the remote
host, you would change the first port number in the argument to `-R` below.

```console
$ ssh -fNT -R8088:localhost:8088 emmanuel@<yourhost>
```

It is a good idea to keep the session into the container running, so you can
easily return when cleaning up.

## Accessing the Service

Now, login to the remote SSH host. You should be able to access the service
running in the container directly under the port `8088`. You can check that it
works using something like `nc` for example:

```console
$ nc -v localhost 8088
```

Provided that you have a working Docker environment on that host, you can even
use the port for developing from **within** a container running on that remote
SSH host. This involves sharing the host network with the container and,
perhaps, mounting your development directory into the container. Running the
following command would give you an Alpine based container with your
development directory mounted on `/data` and where you are able to access
`8088` from the container that you wanted to debug.

```console
$ docker run -it --rm --network=host -v `pwd`:/data alpine
```

## Cleaning Up

To clean up, apart from leaving the debugging container described in the
previous section, you should return to the initial container, kill the `ssh`
process running there are remove the ssh client package from the distribution.