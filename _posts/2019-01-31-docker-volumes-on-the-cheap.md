---
layout: post
title: "Docker Volumes on the cheap"
---

Docker [volumes] plugins have an [API]. When creating a volume plugin, in
addition to implement the API, you will also have to implement the volume
[API][vAPI]. On the other hand, there are a large number of [fuse]-based
implementations for remote storages of various sorts. This post is about
leveraging these implementations as pseudo-volumes, while performing the mount
in a container. This comes at the cost and complexity of sharing a well-known
directory on the host between the container that will perform the mount, with
the container(s) that use the mount.

  [volumes]: https://docs.docker.com/storage/
  [API]: https://docs.docker.com/engine/extend/plugins_volume/#volume-plugin-protocol
  [vAPI]: https://docs.docker.com/engine/extend/#developing-a-plugin
  [fuse]: https://github.com/libfuse/libfuse

## Capabilities

The first step to manage and understand is how to give away enough rights to the
container that will perform the mount so that other processes on the host
(outside that container) will be able to access the files and directories. This
can be achieved through using the following options for docker [run]:

* Give the fuse device to your container through `--device /dev/fuse`
* Raise its capabilities to bypass some of the security layers through:
  `--cap-add SYS_ADMIN` and `--security-opt "apparmor=unconfined"`
* Mount the local host file system into the container in a way that it can be
  shared back with other processes and containers using
  [`rshared`][propagation].

  [run]: https://docs.docker.com/engine/reference/run/
  [propagation]: https://docs.docker.com/storage/bind-mounts/#configure-bind-propagation
  
## Unmounting

You also want to properly unmount when the container gracefully terminates. This
is achieved through a combination of [tini] and proper `trap` in the shell. The
reason for this is that your are likely to interface the [fuse] implementation
through some sort of [entrypoint]

  [tini]: https://github.com/krallin/tini
  [entrypoint]: https://docs.docker.com/engine/reference/builder/#entrypoint

### tini

Use [tini] as the main [entrypoint] in your `Dockerfile` and arrange to give it
the `-g` option so that it properly propagates signals to the entire mounting
system in the container. [tini] exists as an Alpine [package] which can be
helpful in keeping down the size of the image.

  [package]: https://pkgs.alpinelinux.org/package/edge/community/x86_64/tini

### Cleanup

In order to clean properly, you can arrange for a shell to be called at the end
of your [entrypoint] implementation, perhaps after having checked that the mount
performed as it should. This shell will trap the `INT` and `TERM` signals,
unmount the volume (lazily) and propagate these to the mount process. Without
[tini], you would never have received these signals, by Docker construction and
design.

## Examples

I have made available two example images following these principles:

* [webdav-client] mounts WebDAV resources and leverages the full potential of
  [davfs2], being able to leverage all the options supported by [davfs2]. This
  is in contrast to the [davfs] volume which also uses [davfs2] under the hood,
  but misses some of the configuration options.
* [s3fs] mounts a remote S3 bucket using the fuse [implementation] with the same
  name. It supports all the official versions of the original fuse projects
  through [tags]

  [webdav-client]: https://cloud.docker.com/u/efrecon/repository/docker/efrecon/webdav-client
  [davfs2]: http://savannah.nongnu.org/projects/davfs2
  [davfs]: https://github.com/fentas/docker-volume-davfs
  [s3fs]: https://cloud.docker.com/u/efrecon/repository/docker/efrecon/s3fs
  [implementation]: https://github.com/s3fs-fuse/s3fs-fuse
  [tags]: https://cloud.docker.com/repository/docker/efrecon/s3fs/tags
