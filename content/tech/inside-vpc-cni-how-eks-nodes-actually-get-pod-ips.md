---
title: "Inside VPC CNI: How EKS Nodes Actually Get POD IPs"
date: 2026-09-25T00:31:00.000-07:00
author: Sitesh Pattanaik
---
Most EKS users treat the pod networking as magic and an abstraction mostly: a pod gets scheduled, it gets a real VPC IP, traffic flow without deeper understand of how things work under the hood. In this blog post, we will de-mystify that by understanding the VPC CNI's daemon - ipamd and why the pods don't stall behind an actual EC2 call.

### CNI plugin vs ipamd
"vpc cni" is mostly considered a single thing, but actually there are two things.
1. CNI plugin binary (aws-cni) : invoked directly by the kubelet on each node on pod create/delete (`cni add`, `cni del`). it's job is per-pod network plumbing.
2. ipamd daemon : runs as a daemonset in every node. it's job is supply management - talking to ec2 to attach/detach eni, assign/unassign secondary ips, pre-warmed ip pool etc.

The CNI plugin talks to ipamd over a local gRPC socket on the same node.

```
Note: previous to 1.24, kubelet directly invoked the CNI plugins via it's own. On current EKS clusters, it's the container runtime (containerd) via CRI, that invokes the CNI plugin
```


### Primary ENI and the node boot sequence
- every ec2 instance launch with one primary eni - `eth0`
- attached by ec2 at instance launch before k8 or cni in picture
- this ENI creates a primary private IP for the node to consume

#### Boot sequence
- ec2 launches the instance with the primary eni attached
- kubelet starts and registers the node with the api-server. the node has `NotReady` taint as the networking is not ready.
- `aws-node` (ipamd + CNI plugin) gets scheduled on the node as they tolerate the NotReady taint to bootstrap the networking.
- **ipamd initializes and does it first pool warm-up**: it reads the instance-type limits from ec2 metadata (max enis, max ips per eni etc), checks the current state of the primary eni, and calculates how many secondary ips needed to satisfy the current configuration and then makes ec2 calls to get the rest ips.
- the node goes to `Ready` as soon as some configured number of ips are allocated, not when the whole warm-pool is met. warming continues in the background with ipmad's reconcile loop.
- every new pod on the node triggers a `CNI ADD` which eventually asks ipamd for a free IP and the pod gets the networking.
- the ipamd controll loop makes sure the ip pool is warmed up to meet the required config.

### Warm pool knobs
- `WARM_IP_TARGET` : always keep atleast N free ips ready. it's fully **bi-directional** that means if there are way too many free ips, the ips get un-assigned based on this config.
- `MINIMUM_IP_TARGET`: a **one-directional** floor on total allocated IPs. It doesn't care about free ips, it just prevents ipamd to shrink below this number in case they become available.
- `WARM_ENI_TARGET`: the older and coarser knob, where keep N full spare ENIs attached (default 1 if `WARM_IP_TARGET` is unset).

### How ipamd actually gets more secondary IPs
When the free pool drops below the target, ipamd doesn't blindly request WARM_IP_TARGET in one shot. It runs a bounded, iterative algorithm.

It then walks through **already-attached ENIs**, one at a time and tries to get it to full before allocating new ENI.

So, if all the ENIs are at capacity and the ip-pool isn't still met, it makes a call to EC2 to create a secondary ENI and get IPs from that.

note: this process goes in the background always.

### How ipamd knows current ENI/IP state
ipamd maintains an in-memory cache of the current state as well as the value from EC2 metadata from it's initial calls

### Takeaways
1. pod IP assigned is fast because expensive EC2 call is decouple and pre-warmed
2. WARM_IP_TARGET and MIN_IP_TARGET solve different problems. one is a bi-directional elastic buffer vs a one-directional pre-scaled floor.
3. new ENI creation is a two step trigger: all ENI are at capacity and still shortfall.
4. subnet selection for ENI creation is three phased
    - default (inherit primary)
    - enhanced discovery (opportunistic pool)
    - custom networking
5. ipamd state is locally cached and checkpointed.
6. vpc-cni install/upgrades never touches kubelet or containerd config.


















































