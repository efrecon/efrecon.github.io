---
layout: post
title: "Git LFS on Ubuntu"
---

Git [LFS] enables the storage of large files out of the main repository,
replacing them by an index and reference to remote storage instead.

  [LFS]: https://git-lfs.github.com/

## Installing on Ubuntu

To install LFS on Ubuntu 16.04+, perform the following. This is almost as
advertised on the LFS home page and at [PackageCloud], with the slight tweak
that you need to actually install the extra package, since the `bash` script
will only add the repository.

  [PackageCloud]: https://packagecloud.io/github/git-lfs/install#bash-deb

```console
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
git lfs install
```

## Why

My repository of [tclkits] uses LFS storage to store binaries that were
(cross-)compiled by the excellent [KitCreator]. This permit other projects to
depend on those binaries in a lightweight form. For an example, have a look how
this [make] script uses a [contract] to download these very binaries into a
location that is under the git repository, but kept out of revision control.

The rationale for hosting those externally to the [concocter] project is to be
able to offer the binaries and downloads as a service to the community as there
seem to be few remotely hosted binaries with support for TLS.

  [tclkits]: https://github.com/efrecon/tclkit/
  [KitCreator]: http://kitcreator.rkeene.org/kitcreator
  [make]: https://github.com/efrecon/concocter/blob/master/make/make.tcl
  [contract]: https://github.com/efrecon/concocter/blob/master/make/bin/bootstrap.dwl
  [concocter]: https://github.com/efrecon/concocter/
