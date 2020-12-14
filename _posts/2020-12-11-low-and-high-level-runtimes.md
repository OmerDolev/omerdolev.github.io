---
title: Low and High Level Container Runtimes (K8s series - #3)
author: Omer Dolev
date: 2020-12-14 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

![runtimes_pic](/assets/img/low-and-high-level-runtimes-1.png "Title")

Let's have a short dive into the world of container runtimes, talking about low-level and high-level runtimes.  
One of the most common low-level container runtimes is [runc](https://github.com/opencontainers/runc) (which by the way, is written in GoLang).

For example, there's [systemd-nspawn](https://wiki.archlinux.org/index.php/Systemd-nspawn) which is a tool that resembles the chroot command
(which changes your root dir) from a user POV, but actually, it enables runnning a command or even an OS in a light-weight namespace container.
Since it's a container, it fully virtualizes the FS (which unlike chroot, that just **changes** your root dir), as well as the hostname,
process tree and various IPC subsystems.

