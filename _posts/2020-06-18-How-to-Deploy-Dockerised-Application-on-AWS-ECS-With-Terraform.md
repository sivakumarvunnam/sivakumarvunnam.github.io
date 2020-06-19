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


## Prerequisites

* An AWS account
* Node installed
* Docker installed and some experience using it
* Terraform installed

## Step 1 – Start your vagrant box

First, run the following commands to create and navigate to our application’s directory:

```
$ vagrant ssh master
```
```
vagrant@sivakumarvunnam:~$ mkdir node-docker-ecs
vagrant@sivakumarvunnam:~$ cd node-docker-ecs/
```
Next, create an ``npm`` project and Install ``Express``:

```
vagrant@sivakumarvunnam:~/node-docker-ecs$ npm init --y
vagrant@sivakumarvunnam:~/node-docker-ecs$ npm install express
```
Create an index.js file with the following code:

```
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => res.send('Hello, This is SivakumarVunnam!'))

app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

The app can then run with below command && You should see your app at ``http://localhost:3000/``:

```
vagrant@sivakumarvunnam:~/node-docker-ecs$ node index.js
```
## Step 2. Dockerize the Node App

Create a ``Dockerfile`` in your project directory and populate it with the following code:
```
# Use an official Node runtime as a parent image
FROM node:12.7.0-alpine

MAINTAINER sivakumarvunnam1@gmail.com

# Set the working directory to /app
WORKDIR '/app'

# Copy package.json to the working directory
COPY package.json .

# Install any needed packages specified in package.json
RUN yarn

# Copying the rest of the code to the working directory
COPY . .

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Run index.js when the container launches
CMD ["node", "index.js"]
```


---
