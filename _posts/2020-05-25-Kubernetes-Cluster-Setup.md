---
layout: post
title: "Kubernetes Cluster Setup on ubuntu-18-04 and 16-04 using Kubeadm"
author: "Sivakumar Vunnam"
excerpt: "In this post to Install kubernetes on Ubuntu 18.04 and 16.04 we are going to create Kubernetes cluster along with kubeadm on Ubuntu 18.04"
keywords: kubeadm, cluster, kubernetes, DevOps
draft: false
---
kubeadm is command line tool to interact with kubernetes cluster. Kubeadm relies on fast deployment feature of Docker and running the system services of kubernetes master and etcd server as container.

We are going to use two nodes

* Master node – Ubuntu 18.04
* Worker node – Ubuntu 18.04

## Prerequisites

Reading time is about 20 minutes
Vagrant 2.2.7 or latest – For installation instruction [Click here](https://www.vagrantup.com/downloads.html)
VM VirtualBox – For installation instruction [Click here](https://www.virtualbox.org/wiki/Linux_Downloads)

## Step 1 – Start your vagrant box

As a minimum requirement for kubernetes installation we need –

* Master Node – 2 cpus, 2 GB Memory
* Worker Node – 1 cpu, 1 GB Memory

Use following Vagrantfile to spin up your vagrant box:
Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box_download_insecure = true    
    master.vm.box = "hashicorp/bionic64"
    master.vm.network "private_network", ip: "100.0.0.1"
    master.vm.hostname = "master"
    master.vm.provider "virtualbox" do |v|
      v.name = "master"
      v.memory = 2048
      v.cpus = 2
    end
  end

  config.vm.define "worker" do |worker|
    worker.vm.box_download_insecure = true 
    worker.vm.box = "hashicorp/bionic64"
    worker.vm.network "private_network", ip: "100.0.0.2"
    worker.vm.hostname = "worker"
    worker.vm.provider "virtualbox" do |v|
      v.name = "worker"
      v.memory = 1024
      v.cpus = 1
    end
  end

end



---
