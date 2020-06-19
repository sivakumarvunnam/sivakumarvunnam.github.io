---
layout: post
title: "Deploy a Dockerised Application on AWS ECS With Terraform"
author: "Sivakumar Vunnam"
excerpt: "Install terraform and create a simple Node app to having it containerized, load-balanced, and deployed using Docker, AWS ECS With Terraform"
keywords: terraform, ecscluster, ecs, DevOps, AWS
draft: false
---
In this post, I will go through the process of deploying a Node app on AWS ECS with Terraform.

## Overview

We will follow these steps:

* Create a simple Node app and run it locally.
* Dockerize the Node app.
* Create an image repository on AWS ECR and push the image.
* Create an AWS ECS cluster.
* Create an AWS ECS task.
* Create an AWS ECS service.
* Create a load balancer.

The technologies used in this guide are:

* ``Amazon ECS`` — a fully managed container orchestration service
* ``Amazon ECR`` — a fully-managed Docker container registry
* ``Terraform`` — an open-source infrastructure as code tool



---
