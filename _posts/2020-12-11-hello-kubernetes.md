---
title: Hello Kubernetes
author: Omer Dolev
date: 2020-12-11 10:10:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes]
---

Kubernetes is a very big subject. I know people that have been working with it for years, and still dont have a good grasp on what's going on in certain parts.
In the next few posts (a Kubernets series) I will try to illustrate what Kubernetes is and what is it's objective as I see it. I will delve into different concepts as we continue on.

**_NOTE:_** From now on I will mostly write K8s instead of Kubernetes

## What is Kubernetes?

**_Fun Fact:_** The name Kubernetes is from Greek, meaning helmsman or pilot (see K8s docs).

To find an answer to that question you can visit K8s docs, but if you are already here then, K8s is summed up most of the times as a container orchestrator.
When you look at it pragmatically it's what most of the job of K8s is, running containers. But in reality it's much much more than that.

There are many different factors embedded in the notion of running containers, and K8s let's you control that if you like. Making things flexible when needed but it also
enables automation. 

We can start understanding things if we look at the state things were before it container orchestration solutions were common. We are actually talking about how things were 
deployed and running over time.

##### Traditional Deployments

Running applications on physical servers. There are lots of problems with this approach which will be described later (of course it's easy to discern retro), but for now let's continue.

##### Virtualized Deployments

Virtualization is a way to run multiple virtual machines (VMs) on one physical server. This is actually running multiple Operating systems (Linux, Windows, Embedded or whatever) that are independent of one another. This was the time when deploying virtual machines runnning the different applications and services was common.

##### Containerized Deployments

Deployment of services and applications as containers. We will have a post about what are containers exactly. For now let's think about them as very lightweight VMs. This is the present, in which deployment of application on containers is very common.


So now in a time where running application is mainly done by containers, there is something very important to keep in mind, and it's the availability of your application.
When an application expriences a fault or a panic in the [Traditional Deployments](#traditional-deployments) era, there is probably monitoring for it, the relevant team would be alerted and would go straight to the server to fix, and later will try to understand what happened (in this case most of fixes are restarts).

Bear in mind, that while the application was down, there was one less endpoint that could handle traffic or do some workload (might harm the availability of the service or make things run slower or for longer).
