---
layout: post
title: "Plexgear AC600 Nano"
---

I recently bought a [Plexgear AC600 Nano](https://www.kjell.com/se/sortiment/dator-natverk/natverk/tradlost-natverk/tradlosa-natverkskort/plexgear-tradlost-usb-natverkskort-433-mb-s-p61345)
USB wifi dongle to bring some life to an old Intel NUC that was lying around
unused. Unfortunately, and unlike as advertised, it does not work under the
latest version of Ubuntu. The linux [drivers](https://www.kjell.com/se/.mvc/Document/Zip?id=a446cdf9-f637-41b2-83a7-a89700fb816a)
provided by the vendor do not work with the latest version of the kernel.

Inside the wifi is a RealTek chipset 8821AU. Here is a rough guide for making
it working with the latest kernel version. These have been tested with a beta
of Ubuntu 18.04 LTS, but should be ok for prior versions as well:

1. Prepare for being able to compile additional stuff for your kernel: `sudo
   apt install dkms build-essentials linux-headersXXX` where XXX matches the
   kernel that you have.
2. Install `git` if you don't already have it, you might want to run this from
   some git UI if you prefer.
3. Clone this [repository](https://github.com/abperiasamy/rtl8812AU_8821AU_linux.git).
   Note that there are a number of "competing" repos at github trying to solve
   the same problems, and not all of them seem to be working.
4. Change directory to the one of the repository above.
5. Issue `sudo make -f Makefile.dkms install`.
6. Load the driver `sudo modprobe -i rtl8812au`.
7. Verify that have a new network interface: `sudo lshw -c network`.

You should now be able to connect to any nearby wifi using the regular gnome
tools (or from the command-line if you prefer). These instructions install
DKMS so your drivers will be recompiled for each (new) version of the kernel.