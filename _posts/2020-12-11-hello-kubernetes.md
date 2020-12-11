---
title: Hello Kubernetes
author: Omer Dolev
date: 2020-12-11 10:10:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes]
---

Kubernetes is a very big subject. I know people that have been working with it for years, and still dont have a good grasp on what's going on in certain parts.
Here I will try to illustrate what Kubernetes is and what is it's objective as I see it. I will delve into different concepts as we continue on.

**_NOTE:_** From now on I will write K8s instead of Kubernetes

## What is Kubernetes?

To find an answer to that question you can visit K8s docs, but if you are already here then, K8s is summed up most of the times as a container orchestrator.
When you look at it pragmatically it's what most of the job of K8s is, running containers. But in reality it's much much more than that.

There are many different factors embedded in the notion of running containers, and K8s let's you control that if you like. Making things flexible when needed but it also
enables automation. 

We can start understanding things if we look at the state things were before it container orchestration solutions were common. Well the state was 
