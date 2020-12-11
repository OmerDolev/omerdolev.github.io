---
title: Containers (K8s series - #2)
author: Omer Dolev
date: 2020-12-11 12:40:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

Containers are awesome. At first they sound quite simple, but they enable many things that weren't possible before. Here I will explain what are containers in a bit of a deep-dive so you can truly understand what they are.

Containers are applications that are abstraced from the environment in which they run. This is the high-level explanation, which is of course correct but it's a bit vague and general. It is noteworthy that this abstraction is very useful for allowing applications to be deployed easily and consistently (that is regardless what machine the container runs on). What's more interesting is how containers are actually implemented (at least in Linux) and just how different they are from VMs.
Understanding how they are implemented in Linux will give you an idea about what they are and will help you understand the whole thing better as we move on to talk about K8s.

## Linux Containers (LXC)

The goal of containers is running workload decoupled from the underlying environment. Even though it's almost totally decoupled there is one thing that ties the container with the underlying machine, the machines **kernel**.

So Linux containers is an operating system virtualization mechanism for running multiple isolated Linux systems (which we call containers) on a host using a single Linux kernel.
This is done, using a feature of the Linux kernel called "Namespaces".

### Linux Namespaces

If you're familier with the term "namespaces" from programming (in GoLang for example it's called a "package"), it's a logical seperate area containing objects isolated from other areas in our code. 

In Linux, a Linux namespace is a logical seperation of operating system resources. In Linux every namespace has a type which determines the type of resources this namespace can contain. For example, a network namespace will contain network interfaces and sockets, a UTS (Unix Timesharing System) namespace contains seperate hostname and domain name, an mnt (or mount) namespace is a set filesystem mounts visible within the namespace. There are other namespaces as well, and we will get to some of them.

Linux has system namespaces which are the main namespaces that all the processes in the system live in by default, and each process is bound to one namespace of each type.
When running containers we actually run a Linux process in namespaces other than the system ones, which means new namespaces for the process are created, and the process is attached to these namespaces.

Let's try, for example, playing with the network namespace. Let's start by viewing the existing ones and creating a new one:

```bash
lsns -t net # -t is namespace type

```
