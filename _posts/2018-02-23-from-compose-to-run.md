---
layout: post
title: "From Compose to Run"
---

Docker [compose](https://docs.docker.com/compose/overview/) provides
higher-level abstractions to bind together containers for running a given
project. The file format has
[evolved](https://docs.docker.com/compose/compose-file/compose-versioning/)
considerably and has become the base for deploying application stacks in [Swarm
Mode](https://docs.docker.com/engine/swarm/).

Sometimes, it might be necessary to manually recreate a "base" container with a
slightly modified set of arguments/options. For these cases, my
[run.tpl](https://gist.github.com/efrecon/8ce9c75d518b6eb863f667442d7bc679) gist
might come to the rescue. Provided a running container on the host, it is able
to output a `docker run` command that would recreate exactly the same container.
Run has a [wealth](https://docs.docker.com/engine/reference/commandline/run/) of
options and the current implementation focuses on the main ones only.

## Example

In order to run this blog locally when writing posts or tweaking appearance, I use this [image](https://github.com/Starefossen/docker-github-pages) and the following command:

```console
docker run -t --rm -v "$PWD":/usr/src/app -p "4000:4000" --name site starefossen/github-pages
```

Provided you have the gist available at `./run.tpl`, running the following
[inspect](https://docs.docker.com/engine/reference/commandline/inspect/) command
with my gist will output back a run command.

```console
docker inspect --format "$(<run.tpl)" site
```

The output would look like the following, a command that is very similar to the original one and where underlying defaults have been streamlined.

```console
docker run \
    --name=/site \
    --env="PATH=/usr/local/bundle/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    --env="RUBY_MAJOR=2.5" \
    --env="RUBY_VERSION=2.5.0" \
    --env="RUBY_DOWNLOAD_SHA256=1da0afed833a0dab94075221a615c14487b05d0c407f991c8080d576d985b49b" \
    --env="RUBYGEMS_VERSION=2.7.6" \
    --env="BUNDLER_VERSION=1.16.1" \
    --env="GEM_HOME=/usr/local/bundle" \
    --env="BUNDLE_PATH=/usr/local/bundle" \
    --env="BUNDLE_BIN=/usr/local/bundle/bin" \
    --env="BUNDLE_SILENCE_ROOT_WARNING=1" \
    --env="BUNDLE_APP_CONFIG=/usr/local/bundle" \
    --env="NODE_MAJOR=6" \
    --env="GITHUB_GEM_VERSION=177" \
    --env="JSON_GEM_VERSION=1.8.6" \
    -p 0.0.0.0:4000:4000/tcp \
    --network "bridge" \
     \
    --volume="/home/emmanuel/dev/projects/efrecon.github.io:/usr/src/app" \
    --log-driver="json-file" \
    --restart="no" \
    -t \
    "starefossen/github-pages" \
    "/bin/sh" "-c" "jekyll serve -d /_site --watch --force_polling -H 0.0.0.0 -P 4000"
```