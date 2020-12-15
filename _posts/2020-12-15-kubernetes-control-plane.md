---
title: Kubernetes Control Plane(K8s series - #5)
author: Omer Dolev
date: 2020-12-15 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

<img src="/assets/img/kubernetes-control-plane-4.png" alt="kubernetes-control-plane" align="middle" height="300" width="300"/>

In K8s, as in many other products the architecture is a manager/master and worker nodes architecture.

I'm not going to cover everything in just one post, so for now let's focus on the control plane.  
If you needed to manage something like this, orchestrating containers, in a very large scale (hundreds to thousands of containers), how would you do it?

That's a very big question. But, let's break it down to simpler objectives or tasks.  

Let's start with the basis for this application. The first thing we might ask ourselves is how would this application know what to do? When to do it? And what to do it to?  
At the lowest level, a good way to start, is to somehow know the objective (the desired state), and find our way of getting to this desired state. Basically we need a place to hold our objective,
components will be able to check what the desired state is and then decide what actions need to be taken to get us there.

OK then! This application would have a part that does the work, and it needs a place for data.  
We will need a "single source of truth" (if we have more than one place where the desired state resides and somehow these different places hold different states, we will have chaos).
This single source of truth will have to be accessible to every component.
Also, to handle such scale, it needs to be distributed because we might need many worker nodes, doing loads of actions, also the control plane parts are going to perform many administrative actions as well.

There is another point here. Let's say we got our database, if many different components will have to perform operations on it, then we will be compelled not to only have a logic in each component that connects to the database, we will also have to make sure that the operations are valid (so we don't have any corruption).  
Doing these validations and support large operations scale is not an easy task.
So, we should also have some kind of a gateway, via which components can access the data. It's easier to validate data, manage, and control.
In addition,  It's also good if we have a standard way of interactions between components, so we should maybe consider having RESTful components.

<img src="/assets/img/kubernetes-control-plane-2.png" alt="kubernetes-control-plane" align="middle"/>

That's where ETCD and the API server of K8s come into play. ETCD is a RESTful hierarchical distributed key-value datastore that can handle large scales (highly available), supports watching (watching for changes of entries)
and secure connections.

**_Fun Fact_**: ETCD uses the raft census algorithm to create a quorum for leader election. The leader is the member through which writes are performed to ensure consistency between member's data.
I will have a post about raft, as it's a cool algorithm that many tools use.

The API server is the gateway to the ETCD, all operations and changes to the desired state are going through the API server whose job is not only to perform them but to also, to validate, ensure the standardization
of data and structures and check authentication and authorization to perform such actions. It's also designed to be scaled up horizontally, to support high traffic.

Hurray! We have our truth source and the gateway to it :)

What we need now is the tools that will actually make the actions.  
What actions are we talking about though? Well...  
Checking the desired state and the actual state and bringing the actual state closer to the desired one.

Also, We need to know what we are going to manage.  
For all that we have the K8s resources presented below:

<img src="/assets/img/kubernetes-control-plane-3.png" alt="kubernetes-control-plane" align="middle" height="500" width="850"/>

And the rest of the components in the control plane: the controller manager, and the scheduler.  
The controller manager (concisely) is responsible for managing controllers (you might think I am joking but that's true). The parts really doing stuff are the controllers. So what's a controller?

A controller is essentially a loop that checks if there are ways that the desired and actual state are different, then it does what's called *reconcile* which is "bringing the current state to the desired state".
These "reconcile" actions are done differently and independently of one another.

The 2 main components of a controller is an Informer/SharedInformer and a WorkQueue.  
Let's elaborate on the Informer a bit. The Informer is a structure that holds a few things.
* a local **cache** where the controller can watch a list of resources and the changes they experience (whenever an object is deleted, modified or created)
* **resource event handler** that configures an AddFunc (for when an object is added), an UpdateFunc (for when an object is updated) and a DeleteFunc(for when an object is deleted)
* **resync period** which sets an interval for when to trigger the UpdateFunc on the items remaining in the cache. This provides a kind of configuration to periodically verify the current state and make it like the desired state. It's extremely useful in the case where the controller may have missed updates or prior actions failed.

SharedInformer (as it's name implies) is a way to share caches that watch resources. It eliminates duplication of cached resources

To sum up, the overall architecture looks roughly like so:

<img src="/assets/img/kubernetes-control-plane-1.png" alt="kubernetes-control-plane" align="middle"/>
