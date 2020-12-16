---
title: Kubernetes - How It Fits Together (K8s series - #6)
author: Omer Dolev
date: 2020-12-16 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

<img src="/assets/img/how-it-all-fits-together-1.png" alt="how-it-all-fits-together" align="middle" height="300" width="600"/>

## Going to the Roots

There is much much more to talk about the control plane, but that's for more specific posts. Eventually, what we want to do in K8s is actually running workloads in the form of Pods.

What's a Pod? It's components? You can read about it from a high-level perspective in the [official documentation](https://kubernetes.io/docs/concepts/workloads/pods/).  
Rewinding back to the containers post, we remember that Linux containers are not really "containers" they are normal processes, executed using 2 features of the Linux Kernel: Namespaces and Control Groups (cgroups).

Pod is actually taking these namespaces and cgroups and leveraging the to make some cool stuff with "containers". Normally, people see pods like standalone boxes.

<img src="/assets/img/how-it-all-fits-together-1.png" alt="how-it-all-fits-together" align="middle" height="500" />
