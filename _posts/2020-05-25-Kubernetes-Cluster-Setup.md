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

* Vagrant 2.2.7 or latest – For installation instruction [Click here](https://www.vagrantup.com/downloads.html)
* VM VirtualBox – For installation instruction [Click here](https://www.virtualbox.org/wiki/Linux_Downloads)

## Step 1 – Start your vagrant box

As a minimum requirement for kubernetes installation we need

* Master Node – 2 cpus, 2 GB Memory
* Worker Node – 1 cpu, 1 GB Memory

Use following Vagrantfile to spin up your vagrant box:

```
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
```
## Step 2 – Update host files on both master and worker node

master node – SSH into the master node

```
$ vagrant ssh master
```

Add host entry for master as well as worker node

```
vagrant@master:~$ sudo vi /etc/hosts

100.0.0.1 master.kubernetes.com master
100.0.0.2 worker.kubernetes.com worker
```
worker node – SSH into the worker node

```
$ vagrant ssh worker
```
Add host entry for master as well as worker node

```
vagrant@worker:~$ sudo vi /etc/hosts

100.0.0.1 master.kubernetes.com master
100.0.0.2 worker.kubernetes.com worker
```

Test the worker node by sending from master

```
[vagrant@master ~]$ ping worker
PING worker.kubernetes.com (100.0.0.2) 56(84) bytes of data.
64 bytes from wprker.kubernetes.com (100.0.0.2): icmp_seq=1 ttl=64 time=0.462 ms
64 bytes from worker.kubernetes.com (100.0.0.2): icmp_seq=2 ttl=64 time=0.686 ms
```
Test the master node by sending from worker

```
[vagrant@worker ~]$ ping master
PING master.kubernetes.com (100.0.0.1) 56(84) bytes of data.
64 bytes from master.kubernetes.com (100.0.0.1): icmp_seq=1 ttl=64 time=0.238 ms
64 bytes from master.kubernetes.com (100.0.0.1): icmp_seq=2 ttl=64 time=0.510 ms
```
## Step 3 – Install Docker on both master and worker node

You need to install Docker on both the node. So run the following installation command on both the nodes

```
[vagrant@master ~]$ sudo apt-get update && apt install docker.io
```
Enable and start docker

```
vagrant@master:~$ sudo systemctl enable docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.

[vagrant@master ~]$ sudo systemctl start  docker
```
Check the docker service status

```
[vagrant@master ~]$ sudo systemctl status docker

● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-04-23 19:10:59 UTC; 4s ago
     Docs: https://docs.docker.com
```

## Step 4 – Disable the firewall and turnoff the “swapping”

We need to disable firewall as well as swapping on master as well as worker node. Because to install kubernetes we need to disable the swapping on both the nodes

```
vagrant@worker:~$ sudo ufw disable
Firewall stopped and disabled on system startup
```
```
[vagrant@worker ~]$ sudo swapoff -a
```
## Step 5 – Install “apt-transport-https” package

To download the kubernetes and its public we need to install “apt-transport-https” package on both master as well as worker node

```
vagrant@master:~$ sudo apt-get update && sudo apt-get install -y apt-transport-https
```
## Step 6 – Download the public keys

We need to have the public keys for accessing packages on Google Cloud. So run the following command to get the public keys on both master as well as worker node

```
vagrant@worker:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
OK
```
## Step 7 – Add kubernetes repo

As a next step we need to add the kubernetes repo to both master as well as worker node

```
vagrant@worker:~$ sudo bash -c 'echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list'
```
## Step 8 – Install kubernetes

Now after adding the kubernetes repo we need to install the kubernetes on both mater as well as worker node

```
vagrant@master:~$ sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
```
## Step 9 – Enable and Start kubelet

Alright now we have installed the kubernetes, now we need to enable the kubelet support for both master as well worker node

```
[vagrant@master ~]$ sudo systemctl enable kubelet

[vagrant@master ~]$ sudo systemctl start kubelet

```
## Step 10 – Initialize the kubernetes cluster

Okay now we have reach to point where we have done all the prerequisite for initializing the kubernetes cluster.

Let’s run the kubernetes initialization command on only on master

```
[vagrant@master ~]$ sudo kubeadm init --apiserver-advertise-address=100.0.0.1 --pod-network-cidr=10.244.0.0/16

```
Note down kubeadm join command which we are going to use from worker node to join the master node using token.

```
kubeadm join 100.0.0.1:6443 --token nj21cz.iwxl9vhgniksdckz \
    --discovery-token-ca-cert-hash sha256:ebfcfcedc9474c5e8154b433a3376f48ce67a97913c1ce4f7b1635c0ba0cef3d
```

## Step 11 – Move kube config file to current user (only run on master)

To interact with the kubernetes cluster and to user kubectl command, we need to have the kube config file with us.

Use the following command to get the kube config file and put it under working directory.

```
[vagrant@master ~]$ mkdir -p $HOME/.kube
[vagrant@master ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[vagrant@master ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## Step 12 – Apply CNI from kube-flannel.yml(only run on master)
After the master of the cluster is ready to handle jobs and the services are running, for the purpose of making containers accessible to each other through networking, we need to set up the network for container communication.

Get the CNI(container network interface) configuration from flannel

```
[vagrant@master ~]$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
Note – But since we are working on the VMs so we need to check our Ethernet interfaces first.
Look out for the Ethernet i.e. eth1 which has a ip address 100.0.0.1(this is the ip address which we used in vagrant file)

```
[vagrant@master ~]$ ip a s
1: lo: <LOOPBACK,UP,LOWER_UP>
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:bb:14:75 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fb:48:77 brd ff:ff:ff:ff:ff:ff
    inet 100.0.0.1
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP>
```
Now we need to add the extra args for eth1 in kube-flannel.yml

```
[vagrant@master ~]$ vi kube-flannel.yml
```
Searche for – “flanneld”

In the args section add : – –iface=eth1

- --iface=eth1
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1
Apply the flannel configuration

```
vagrant@master:~$ kubectl apply -f kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
```
## Step 13 – Join worker nodes to master(only run on worker)

In the Step 10 we generated the token and kubeadm join command. Now we need to use that join command from our worker node

```
vagrant@worker:~$ sudo kubeadm join 100.0.0.1:6443 --token g2bsw7.5xr3bqc21eqyc6r7     --discovery-token-ca-cert-hash sha256:39b2b0608b9300b3342a8d0a0e9204c8fc74d45b008043a810f94e4f1fb8861f
W0423 19:27:00.344480   18268 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
## Step 14 – Check the nodes status(only run on master)
To check the status of the nodes use

```
[vagrant@master ~]$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   26m   v1.18.2
worker   Ready    <none>   63s   v1.18.2
```
Summary
So this was our beginner tutorial which involves around – 14 Steps to Install kubernetes on Ubuntu 18.04 and 16.04.

With that we can conclude what we have achieved 

Congratulations we have successfully installed kubernetes on Ubuntu 18.04
After installing kubernetes we are successfully join the worker nodes with master.





---
