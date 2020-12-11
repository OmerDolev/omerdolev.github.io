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

