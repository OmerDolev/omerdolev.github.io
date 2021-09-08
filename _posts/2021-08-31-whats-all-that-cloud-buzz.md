---
title: Amazon Web Services
author: Omer Dolev
date: 2021-08-31 16:26:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, aws, cloud]
---


<img src="/assets/img/whats-all-that-cloud-buzz-pic-1.png" alt="whats-all-that-cloud-buzz" align="middle"/>

## What's All That Cloud Buzz?!

One of the reasons so many new start-ups can try to make our life easier is "THE CLOUD".

Let's say you have an apartment, you have your living room, bedroom, bath room, kitchen and accessories. 
Suddenly, you get a deal from your work place, a 3 months vacation (a place eveyone would like to work at) to the caribbeans. During these 3 months, even if you are not effectively using your apartment, you still have to pay rent and additional bills and fees.  
In that case, what many would do is subletting their apartment, which means that they can make some money during their vacation and cover the expenses for their apartment.

Let's get a bit more technical.  
Say you are a company and you have a datacenter that for an irrelevant reason would have to stay inactive unless you somehow let people run workloads there. In that case you can cover expenses and might even get some profit out of it.

This is basically (very basically) what cloud is.  
There are companies that have a vast amount of compute resources. Those companies need lots of resources to run their own services and still they have so much more, that they lend those compute resources to other companies to run their services. It's working so well that there are companies worth Ms and even Bs of $ that run entirely in the cloud (in another company's datacenter).

Another thing that made cloud approachable is the accessability. All you need to do is just create an account, make sure there is a payment method and you can start using the cloud provider resources (providers - such as Google, Amazon and Microsoft - are the those that lend resources to other companies).

The billing model is also quite simple. The general concept is "pay for what you use". For example, if you run a server you pay for the time the server was running. In storage however, because storage is not "temporary" you pay the fare for a certain capacity usage (e.g. per gigabyte).  
Each service has a different billing model based on that.

And this is just the beginning, there are more complicated concepts that enable cost reduction, since you can acheive the same goal with many different architectures and solutions whose costs are different from one another. Things like burst capacity for CPU, Memory and even I/O, enable utilization peak tolerance (so instead of using more servers - that cost more money - you use the same ones, but still cope with the load). Others are reservations (of resources which decreases costs dramatically but requires planning) and spot instances (resource scavenging). But the most comprehensive way to manage costs better is good architecure and practices.
