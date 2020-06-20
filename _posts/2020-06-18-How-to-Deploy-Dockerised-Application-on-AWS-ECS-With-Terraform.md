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
## Step 3. Build and Push the Node App to AWS ECR

Now it is time build and we our container to a container registry service — in this case, we will use ``AWS ECR`` we will use terraform to create our repository. In your directory create a file called main.tf. Populate your file with the following commented code:

```
provider "aws" {
  region                  = "us-west-2"
  shared_credentials_file = "/home/vagrant/.aws/creds"
  profile                 = "default"
}

resource "aws_ecr_repository" "nodeapp_ecr_repo" {
   name = "nodeapp"
}
```
Next, in your terminal, type:

```
terraform init
terraform plan
terraform apply
```
Now we can build and push our Node application image up to this repository. Click on the repository and click View push commands. A modal will appear with four commands you need to run locally in order to have your image pushed up to your repository:

Once you have run these commands, you should see your pushed image in your repository:


## Step 4. Create the  ECS Cluster
ECS has three parts: clusters, services, and tasks.
Next, add this code to your terraform file and redeploy your infrastructure with terraform apply:

```
resource "aws_ecs_cluster" "nodeap_cluster" {
   name = "nodeapp-cluster" # Naming the cluster
}
```
You should then see your new cluster:


## Step 5. Create the First Task

Creating a task is a bit more involved than creating a cluster. Add the following commented code to your terraform script:

```
resource "aws_ecs_task_definition" "my_first_task" {
  family                   = "my-first-task" # Naming our first task
  container_definitions    = <<DEFINITION
  [
    {
      "name": "my-first-task",
      "image": "${aws_ecr_repository.nodeapp_ecr_repo.repository_url}",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000,
          "hostPort": 3000
        }
      ],
      "memory": 512,
      "cpu": 256
    }
  ]
  DEFINITION
  requires_compatibilities = ["FARGATE"] # Stating that we are using ECS Fargate
  network_mode             = "awsvpc"    # Using awsvpc as our network mode as this is required for Fargate
  memory                   = 512         # Specifying the memory our container requires
  cpu                      = 256         # Specifying the CPU our container requires
  execution_role_arn       = "${aws_iam_role.ecsTaskExecutionRole.arn}"
}

resource "aws_iam_role" "ecsTaskExecutionRole" {
  name               = "ecsTaskExecutionRole"
  assume_role_policy = "${data.aws_iam_policy_document.assume_role_policy.json}"
}

data "aws_iam_policy_document" "assume_role_policy" {
  statement {
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ecs-tasks.amazonaws.com"]
    }
  }
}

resource "aws_iam_role_policy_attachment" "ecsTaskExecutionRole_policy" {
  role       = "${aws_iam_role.ecsTaskExecutionRole.name}"
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}
```

---
