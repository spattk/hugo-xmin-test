---
title: "Programming Kubernetes: Chapter 1 - Introduction"
date: "2026-05-16"
tags: ["distributed-systems", "cloud", "programming-kubernetes", "kubernetes", "book-club"]
---

### Programming Kubernetes: Chapter 1 - Introduction

#### what does it mean to program kubernetes

- making kubernetes specific application that interacts with the api-server, queries the state of the resources and/or updates the state
- cloud-native/kubernetes-native: the software is aware that it will be run on a cloud or kubernetes world

#### extension patterns with which we can program kubernetes
- integrates with cloud with cloud-controller-manager
	- cloud providers allow the use of cloud-resources like LB or VMs
- binary kubelet plugins for network, devices, stoage and container runtime
- kubectl plugins
- access extension like dynamic admission control with webhooks
- custom resources and custom controller
- custom api servers
- scheduler extensions to implement own scheduling decisions
- authN with webhooks

#### controllers and operators
- controllers are anything that implement a control loop
- operators == controller + operational knowledge


#### data structures used by controllers
- informers: watch the desired state of resources using scalable patterns
- work queues: the queue that event handlers use to manage retries, ordering of events etc

#### event
- k8 is very heavy event based system
- form of communication from api-servers to informers in controllers via watches
- watches: streaming connection of watch events


#### watch event vs event object
- watch events are sent from api-servers to controllers via informers
- event object is a top level object is a logging mechanism

#### edge vs level triggered
- edge: when there is any updates that happened to the state
- level: constant repolling

kubernetes uses a hybrid where there are continuous resync on top of level based triggers

#### optimistic concurrency
- every controller/operator should work in the world of optimistic concurrency and should implement retry on conflict

#### resource version:
- an etcd version that is used to track the updates to an object