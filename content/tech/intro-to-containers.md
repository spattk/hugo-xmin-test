---
title: intro to containers
date: 2026-06-15T11:55:00.000-07:00
author: Sitesh Pattanaik
---
containers are a pretty interesting eco-system in computer science. to understand some of the simple things like why `containerd` exists outside of the ecosystem, what happened when kubernetes dropped the support for docker in 1.24+ etc, we need to understand a bit of history of containers.

so let's go very basic. 

### container

container is a simplex linux process with two specific kernel features added to it

* namespaces : for network, filesystem isolation
* cgroups: to limit the cpu and memory that the process can use

that's it, there is no magic, no VM, no hypervisor. a container is a process in a box.

### container runtime

the software that's needed to create the box, setting up namespaces, applying the cgroups and starting the process inside

### runc

it's the bottom of the stack for making sure the container happens. 

* developed at docker and open-sourced.
* it's a tiny cli binary
* takes a `config.json` and a root filesystem and gets the container running

### docker era

before all these complexity, it was all docker

* pulled images
* stored images
* setup networking
* ran the container using runc
* managed logs
* exposed cli 

it was monolithic and the only boss, hence no one cared

### kubernetes and 2014+

kubernetes enters and gets into trouble and docker was mostly designed for human use and not directly use by software. so kubernetes faced a lot of challenges there.



furthermore, the complexity to integrate with another container runtime apart from docker isn't a trivial path forward. hence, kubernetes introduced **`container runtime interface (cri)`**

### container runtime interface

a standard api, that provided a plug and play interface. if a new container runtime implements CRI, then kubernetes can talk to it. 

the main challenge here was docker wasn't speaking cri. as it came before the cri existed. as kubernetes was still using docker, so it had to introduce some layer of translation that the docker daemon could understand. and that's how **`dockershims`** was born

### dockershim

so to support the above, dockershim came as part of kubernetes. the flow for kubernetes after cri is this

`kubelet → dockershim → Docker daemon → containerd → runc → container`

### containerd

docker internally separated out it's internal container management into a separate open source component called containerd which was donated to cncf.

it used to manage the following

* pulling and storing images
* managing container lifecycles
* snapshotting filesystems
* talking to runc to actually start containers

long running daemon with clean APIs.

### docker images

Docker the image format and Docker the runtime are two completely separate things. when kubernetes "dropped Docker," it dropped Docker as a *runtime* (the thing that runs containers). it did **not** drop the image format.
