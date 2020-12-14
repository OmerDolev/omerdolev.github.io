---
title: Low and High Level Container Runtimes (K8s series - #3)
author: Omer Dolev
date: 2020-12-14 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

<img src="/assets/img/low-and-high-level-runtimes-1.png" alt="low-and-high-level-runtimes" width="300" height="300" align="middle"/>

Let's have a short dive into the world of container runtimes, talking about low-level and high-level runtimes (which from now will be referred to
as low-level runtimes and high-level runtimes).  
One of the most common low-level container runtimes is [runc](https://github.com/opencontainers/runc) (which by the way, is written in GoLang).

As mentioned in the previous post, conatiners (in Linux of course) are implemented by Linux namespaces and cgroups. Namespaces help you virtualize
the environment while cgroups helps you limit resources consumed by a process. The main responsibility of the low-level runtimes is the creation and 
configuration of such namespaces and cgroups for containers, and then execution inside those namespaces and cgroups (this is the core functionality
of the low-level runtimes, which usually implement more features).

###Let's create a container :)

**_NOTE:_** Based on Ian Lewis [post](https://www.ianlewis.org/en/container-runtimes-part-2-anatomy-low-level-contai)

I am working on Ubuntu, so to control cgroups you need to run the following (all the cg* commands will probably require sudo):

```bash
# install cgroups control commands
sudo apt-get install cgroup-lite cgroup-tools cgroupfs-mount libcgroup1
```

We will use a simple container FS as our base FS for our container:

```bash
# creating a sample container and exporting it's FS to a tmp dir
CID=$(docker create busybox)
ROOTFS=$(mktemp -d)
docker export $CID | tar -xf - -C $ROOTFS
```

We will now create cgroups and limit resources for the container:

```bash
# creating cgroups - cpu and memory
UUID=$(uuidgen)
cgcreate -g cpu,memory:$UUID

# configuring memory limit to 100000000 bytes (100MB)
cgset -r memory.limit_in_bytes=100000000 $UUID
# configuring cpu shares to 512
cgset -r cpu.shares=512 $UUID
# we will use the cfs mechanism here (will be explained in a seperate post)
cgset -r cpu.cfs_period_us=1000000 $UUID
cgset -r cpu.cfs_quota_us=2000000 $UUID
```

OK! We create our container sandbox! Lets execute a command in the container:

```bash
$ cgexec -g cpu,memory:$UUID \
>     unshare -uinpUrf --mount-proc \
>     sh -c "/bin/hostname $UUID && chroot $ROOTFS /bin/sh"
/ # echo "Hello from in a container"
Hello from in a container
/ # exit
```

* The unshare command is used to execute a process in different namespaces

Now to clean up, let's delete our cgroups and tmp dir:

```bash
cgdelete -r -g cpu,memory:$UUID
rm -r $ROOTFS
```

For example, there's [systemd-nspawn](https://wiki.archlinux.org/index.php/Systemd-nspawn) which is a tool that resembles the chroot command
(which changes your root dir) from a user POV, but actually, it enables runnning a command or even an OS in a light-weight namespace container.
Since it's a container, it fully virtualizes the FS (which unlike chroot, that just **changes** your root dir), as well as the hostname,
process tree and various IPC subsystems.

