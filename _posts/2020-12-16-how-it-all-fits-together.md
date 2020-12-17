---
title: Kubernetes - How It Fits Together (K8s series - #6)
author: Omer Dolev
date: 2020-12-16 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

<img src="/assets/img/how-it-all-fits-together-1.png" alt="how-it-all-fits-together" align="middle" height="300"/>

## Going to the Roots

There is much much more to talk about the control plane, but that's for more specific posts. Eventually, what we want to do in K8s is actually running workloads in the form of Pods.

What's a Pod? It's components? You can read about it from a high-level perspective in the [official documentation](https://kubernetes.io/docs/concepts/workloads/pods/).  
Rewinding back to the containers post, we remember that Linux containers are not really "containers" they are normal processes, executed using 2 features of the Linux Kernel: Namespaces and Control Groups (cgroups).

Pod is actually taking these namespaces and cgroups and leveraging them to make some cool stuff with "containers". Normally, people see pods like standalone boxes.

<img src="/assets/img/how-it-all-fits-together-2.png" alt="how-it-all-fits-together" align="middle" height="300" />

But there are cool things we can do with namespaces, and they have quite a flexible functionality.  
Let's create an nginx container and a ghost container in the same namespaces so they are able to talk to each other:

```
# conf file for the nginx
cat <<EOF >> nginx.conf

error_log stderr;
events { worker_connections  1024; }
http {
  access_log /dev/stdout combined;
  server {
    listen 80 default_server;
    server_name example.com www.example.com;
    location / {
      proxy_pass http://127.0.0.1:2368;
    }
  }
}
EOF

# let's create the nginx container
# in the case of the IPC namespace we need to run the first container with shareable IPC mode
docker run -d --ipc="shareable" --name nginx -v "$(pwd)"/nginx.conf:/etc/nginx/nginx.conf -p 8080:80 nginx

# now let's create the ghost container
# notice, we are sharing network, IPC, and PID namespaces with the nginx
# we can share more namespaces or less namespaces as we like
# but for the sake of the example lets share these 3
docker run -d --name ghost --net=container:nginx --ipc=container:nginx --pid=container:nginx ghost
```

After running these commands you can go visit http://localhost:8080/ and see the ghost page behind the Nginx we created.  
But let's break down what we have just done.

The Nginx is a container, living in its own network namespace. So 127.0.0.1 is the loopback address in the network namespace of the container.
Nginx is configured to listen to port 80, and the config says that when we get a request we forward it to http://127.0.0.1:2368, so we actually forward it to a different port (the ghost port).

Then we ran the ghost container and put it in the same net, PID and IPC namespaces as the Nginx container (I shared 3 namespaces even though just sharing the network one would work).

**_NOTE_**: In this example, even though sharing only the network namespace would work (try it yourself), sharing other namespaces (like PID, IPC, etc...) can be very useful for use-cases that require a certain level of IPC (inter-process communication) or data sharing.

So now that the containers are in the same net namespace, the 127.0.0.1 of the Nginx, is actually the 127.0.0.1 of the ghost as well. That's why this forwarding works.

<img src="/assets/img/how-it-all-fits-together-3.png" alt="how-it-all-fits-together" align="middle" />
