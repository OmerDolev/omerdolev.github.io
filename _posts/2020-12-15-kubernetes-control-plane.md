---
title: Kubernetes Control Plane(K8s series - #5)
author: Omer Dolev
date: 2020-12-15 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

In K8s, as in many other products the architecture is a manager/master and worker nodes architecture. In a high level it looks something like this:

I'm not going to cover everything in just one post, so for now let's focus on the control plane.  
If you needed to manage something like this, orchestrating containers, in a very large scale (hundreds to thousands of containers), how would you do it?

That's a very big question. But, let's break it down to simpler objectives or tasks.  

Let's start with the basis for this application. The first thing we might ask ourselves is how would this application know what to do? When to do it? On what to do it to?  
At the lowest level, a good way to start, is to somehow know the objective (the desired state), and find our way of getting to this desired state. Basically we need a place to hold our objective,
components will be able to check what the desired state is and then decide what actions need to be taken to get us there.

OK then! This application would have a part that would do the work, and it needs a place for data.
Also, to handle such scale, it needs to be distributed because we might need many worker nodes, doing loads of actions, also the control plane parts are going to perform many administrative actions as well.

There is another point here. Let's say we got our database, if many different components will have to perform operations on it, then we will have not only have a logic in each component that connects to the database,
we will also have to make sure that the operations are valid (so we don't have any corruption).  
Doing these validations and support large operations scale is not an easy task.
We should also have some kind of a gateway, via which components can access the data. It's easier to validate data, manage, and control.
In addition, we will need a "single source of truth" (if we have more than one place where the desired state resides and somehow these different place hold different states, we will have chaos),
that will be accessible to every component (via the gateway). It's also good if we have a standard way of interactions between components, so we should maybe consider having RESTful components.

That's where ETCD and the API server of K8s come into play. ETCD is a RESTful hierarchical key-value distributed datastore that can handle large scales, supports watching (watching for changes of entries)
and secure connections.

**_Fun Fact_**: ETCD uses the raft census algorithm to create a quorum for leader election. The leader is the member through which writes are performed to ensure consistency between member's data.
I will have a post about raft, as it's a cool algorithm that many tools use.

The API server is the gateway to the ETCD, all operations and changes to the desired state are

<img src="/assets/img/kubernetes-control-plane-1.png" alt="kubernetes-control-plane" align="middle"/>
