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

Let's try, for example, playing with the network namespace. Let's start by viewing the existing ones and creating a new one (you might need *sudo* for those commands):

```bash
omerd@myhost:~$ ip netns list
omerd@myhost:~$ ip netns add nstest
omerd@myhost:~$ ip netns list
nstest
```

Now remember this is a namespace seperated from the host system namespace.
We can now create a veth interfaces pair to connect the two namespaces.

```bash
omerd@myhost:~$ ip link add v-eth1 type veth peer name v-peer1
omerd@myhost:~$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 54:12:14:d1:56:23 brd ff:ff:ff:ff:ff:ff
3: v-peer1@v-eth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 54:12:12:da:56:23 brd ff:ff:ff:ff:ff:ff
4: v-eth1@v-peer1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 54:19:12:1a:58:23 brd ff:ff:ff:ff:ff:ff
omerd@myhost:~$ ip link set v-peer1 netns nstest
```

After this, we can set addresses for the interfaces and bring them up:

```bash
omerd@myhost:~$ ip addr add 10.200.1.1/24 dev v-eth1
omerd@myhost:~$ ip link set v-eth1 up
omerd@myhost:~$ ip netns exec nstest ip addr add 10.200.1.2/24 dev v-peer1      # Notice that running regular ip commands is in the system namespace
omerd@myhost:~$ ip netns exec nstest ip link set v-peer1 up                     # and to run the ip commands inside a namespace you need to add the 
omerd@myhost:~$ ip netns exec nstest ip link set lo up                          # ip netns exec <ns_name> before the command
```

OK! now we have a new net namespace and an interface up with an address of 10.200.1.2, also we brought the loopback interface up in nstest.

Now let's make a route to forward all traffic from nstest to the system namespace i.e. to v-eth1.

```bash
omerd@myhost:~$ ip netns exec nstest ip route add default via 10.200.1.1        # This adds a route in the routing table to be, by default forwarded to v-eth1
```

We are almost there, but for the internet connection we want to enable forwarding in the system namespace and share internet access between the host and nstest.
This is done by enabling forwarding using the iptables interface.

```bash
omerd@myhost:~$ echo 1 > /proc/sys/net/ipv4/ip_forward                       # by default there's a '0' there disabling this
omerd@myhost:~$ iptables -P FORWARD DROP                                     # setting default policy in the FORWARD chain to DROP
omerd@myhost:~$ iptables -F FORWARD                                          # flushing forward rules
omerd@myhost:~$ iptables -t nat -F                                           # flushing nat rules
omerd@myhost:~$ iptables -t nat -A POSTROUTING -s 10:200.1.0/255.255.255.0 -o eth0 -j MASQUERADE
omerd@myhost:~$ iptables -A FORWARD -i eth0 -o v-eth1 -j ACCEPT              # Allowing forwarding between eth0 and v-eth1
omerd@myhost:~$ iptables -A FORWARD -o eth0 -i v-eth1 -j ACCEPT              # both ways
```

Awesome! Now let's try to see if we have a connection:

```bash
omerd@myhost:~$ ip netns exec nstest ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=50 time=48.6ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=50 time=52.1ms
```

**_NOTE:_** This network namespace game was inspired by https://blogs.igalia.com/dpino/2016/04/10/network-namespaces/

GREAT! 

### Back to Containers

So we have an example for the net namespace, now a container is actually attached not only to a different net namespace but also to a different pid namespace (seeing a isolated pid tree), mnt namespace (seeing different mounts), uts namespace (seeing different hostname).
This collection of namespaces for a container, is called a container ***sandbox*** in some places.

Now you might think, wait... what about resources, as in CPU and memory. Containers are using the system resources as any other process, they just see a different environment . When we get to K8s we would also like to control how much resources a workload uses, so for that we have cgroups (Control Groups, which is another Linux kernel feature).

