---
layout: post
title: "All tags of a Docker (Hub) image"
---

The registry behind the Docker [hub] has an [API]. You can use this API to list
out all the tags for a given existing image using code similar to the following.
This heavily deviates from the [original] to enable the use of `curl`
(preferred) or `wget` and without using `jq`, but rather relying on standard
linux command-line tooling.

  [hub]: https://cloud.docker.com/
  [API]: https://docs.docker.com/registry/spec/api/
  [original]: http://www.googlinux.com/list-all-tags-of-docker-image/index.html

```bash
# This is the image that you wish to list the tags for
im="abiosoft/caddy"

if [ -z "$(echo "$im" | grep -o '/')" ]; then
    hub="https://registry.hub.docker.com/v2/repositories/library/$im/tags/"
else
    hub="https://registry.hub.docker.com/v2/repositories/$im/tags/"
fi

# Get number of pages
if [ -z "$(command -v curl)" ]; then
    first=$(wget -q -O - $hub)
else
    first=$(curl -sL $hub)
fi
count=$(echo $first | sed -E 's/\{\s*"count":\s*([0-9]+).*/\1/')
pagination=$(echo $first | grep -Eo '"name":\s*"[a-zA-Z0-9_.-]+"' | wc -l)
pages=$(expr $count / $pagination + 1)

# Get all tags one page after the other
tags=
i=0
while [ $i -le $pages ] ;
do
    i=$(expr $i + 1)
    if [ -z "$(command -v curl)" ]; then
        page=$(wget -q -O - "$hub?page=$i")
    else
        page=$(curl -sL "$hub?page=$i")
    fi
    ptags=$(echo $page | grep -Eo '"name":\s*"[a-zA-Z0-9_.-]+"' | sed -E 's/"name":\s*"([a-zA-Z0-9_.-]+)"/\1/')
    tags="${ptags} $tags"
done

# Once here, the variable tags should contain the list of all tags for the image.
```

This code can be used in [hooks] to write complex build/push instructions. Such
instructions can be used to automatically enhanced a standard image with
additional features in a future-proof way: for every new version of the original
image, watching it will ensure that your enhanced version will get built.

  [hooks]: https://docs.docker.com/docker-hub/builds/advanced/#custom-build-phase-hooks