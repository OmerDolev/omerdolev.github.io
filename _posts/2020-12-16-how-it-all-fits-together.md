---
title: Kubernetes - How It Fits Together (K8s series - #6)
author: Omer Dolev
date: 2020-12-16 09:50:00 +0800
categories: [Blogging, Tutorial]
tags: [writing, kubernetes, containers]
---

<img src="/assets/img/how-it-all-fits-together-1.png" alt="how-it-all-fits-together" align="middle" height="500"/>

## Going to the Roots

There is much much more to talk about the control plane, but that's for more specific posts. Eventually, what we want to do in K8s is actually running workloads in the form of Pods.

What's a Pod? It's components? You can read about it from a high-level perspective in the [official documentation](https://kubernetes.io/docs/concepts/workloads/pods/).  
Rewinding back to the containers post, we remember that Linux containers are not really "containers" they are normal processes, executed using 2 features of the Linux Kernel: Namespaces and Control Groups (cgroups).

Pod is actually taking these namespaces and cgroups and leveraging them to make some cool stuff with "containers". Normally, people see pods like standalone boxes.

<img src="/assets/img/how-it-all-fits-together-2.png" alt="how-it-all-fits-together" align="middle" height="500" />

But there are cool things we can do with namespaces, and they have quite a flexible functionality.  
Let's create an nginx container with the following conf file:

```
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

# now let's run the container
docker run -d --name nginx -v `pwd`/nginx.conf:/etc/nginx/nginx.conf -p 8080:80 nginx
```
