---
layout: post
title: "Git LFS on Ubuntu"
---

Git [LFS](https://git-lfs.github.com/) enables the storage of large files out of
the main repository, replacing them by an index and reference to remote storage
instead.

To install LFS on Ubuntu 16.04+, perform the following. This is almost as
advertised on the LFS home page and at [Package
Cloud](https://packagecloud.io/github/git-lfs/install#bash-deb), with the slight
tweak that you need to actually install the extra package, since the `bash`
script will only add the repository.

```console
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
git lfs install
```