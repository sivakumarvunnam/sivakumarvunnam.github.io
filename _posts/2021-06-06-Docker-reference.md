---
layout: post
title: "Docker Reference Guide"
author: "Sivakumar Vunnam"
excerpt: "Docker is an open platform for building, shipping and running distributed applications. It gives programmers, development teams and operations engineers the common toolbox they need to take advantage of the distributed and networked nature of modern applications. In other words , Docker is an abstraction on top of low-level operating system tools that allows you to run one or more containerized processes or applications within one or more virtualized Linux instances."
keywords: Docker, Docker-Compose, DevOps, K8S
draft: false
---
### Docker images:
The Docker Flow, the fundamental concept, in Docker, it all begins with an image. An image is every file that makes up just enough of the operating system to do what you need to do.

Traditionally you'd install a whole operating system with everything for each application you do. With Docker you pair it way down so that you have a little container with just enough of the operating system to do what you need to do, and you can have lots and lots of these efficiently on a computer.

### List All Docker Images:
command: docker images

```List running images:
docker ps -a
To get last images runned:
docker ps -l
```



---
