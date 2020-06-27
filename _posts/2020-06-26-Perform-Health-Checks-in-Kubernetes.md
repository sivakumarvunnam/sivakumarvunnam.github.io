---
layout: post
title: "How to Perform Health checks in Kubernetes (K8s)"
author: "Sivakumar Vunnam"
excerpt: "Kubernetes provides a health checking mechanism to verify if a container in a pod is working or not working."
keywords: Kubernetes, DevOps
draft: false
---
Kubernetes provides a health checking mechanism to verify if a container in a pod is working or not working

![](https://miro.medium.com/max/1400/0*J_K_oFbZesKVMoDY.gif)


Health Checks source Thanks to (https://wideops.com/)
Kubernetes gives you three types of health checks performed by the kubelet. They are:
* Startup Probe
* Liveness Probe
* Readiness Probe

# Startup Probe
Whenever we are dealing with physical/Legacy apps those may require extra startup time at first initialisation. In this cases we have a tendency to established a startup probe with constant command, protocol or TCP check, with a failureThreshold periodSeconds long enough to hide the more severe case startup time.
```
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```
# Liveness Probe

Liveness probe checks the status of the container (whether it is running or not). If livenessProbe fails, then automatically container move on with its restart policy

![](https://miro.medium.com/max/1400/0*U8l_OiseiAaGJ_6n.gif)

Liveness Probe source Thanks to (https://wideops.com/)

# Readiness Probe

Readiness probe checks whether your application is ready to serve the requests. When the readiness probe fails, the pod’s IP is removed from the endpoint list of the service.

![](https://miro.medium.com/max/1400/0*kmIsS43VM71shv_1.gif)

Readiness Healthcheck source Thanks to (https://wideops.com/)

There are three types of actions kubelet performs on a pod, which are:

* Executes a command inside the container
* Checks for a state of a particular port on the container
* Performs a GET request on container’s IP

---
