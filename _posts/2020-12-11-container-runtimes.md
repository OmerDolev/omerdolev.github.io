---
title: Container Runtimes (K8s series - #3)
author: Omer Dolev
date: 2020-12-11 16:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

Remember all the things we now know about containers?
So every time we want a container we need to create different namespaces and then spawn the process, and when the process exits we need remove them all
and do some cleanup? NO...

This is what container runtimes are for. As you will see in a moment there are many runtimes, and they are usually divided into low-level and high-level runtimes.
One of the most famous container runtimes is Docker. But since Docker's release many changes took place in this field on container runtimes.

## Let's start some containers! oh... wait...

Before we continue we need to understand how a runtime systemizes container creation. In the [previous post](https://omerdolev.github.io/posts/containers/)
we saw that containers are actually processes that live in different namespaces than the system ones. But there is one important thing we didn't mention.

If a container is a process, and the process spawns in a different mnt namespace (a different filesystem basically), don't we need the executable to be in the
filesystem that resides in that mnt namespace? 

The answer is yes. That's what ***images*** are for. A container image is usually a zipped file that contains a filesystem with an executable and all the files
required for that executable to run, and some metadata for this image (for instance, maybe this executable should be run with arguments, maybe some environment
variables should be set). All this data and the filesystem is called an image.

So let's take Docker as an example.
