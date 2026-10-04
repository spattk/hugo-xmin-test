---
title: Building blocks of Custom Kubernetes Operators and Controllers
date: 2026-10-04T06:09:05.417Z
author: Sitesh Pattanaik
---
![]()

#### what is controller-runtime?

controller-runtime is official set of Go libraries used to build custom operators and controllers in kubernetes.

#### How does a sample custom controller looks like in kubernetes

```
# main.go

package main

import(
 "context"
 "sigs.k8s.io/controller-runtime/pkg/client"
 ctrl "sigs.k8s.io/controller-runtime/pkg/manager"
)

func main() {
  cfg := ctrl.GetConfigOrDie()
  mgr, err := ctrl.NewManager(cfg, ctrl.Options{})
  handleErr(err)
  
  r := &PodCounter{Client: mgr.GetClient()}
  
  err = r.SetupWithManager(mgr)
  handleErr(err)
  
  err = mgr.Start(ctrl.SetupSignalHandler())
  handleErr(err)
  
}

func (r *PodCounter) SetupWithManager(mgr ctrl.Manager) error {
  return ctrl.NewControllerManagedBy(&mgr).
           For(&corev1.Pod{}).
           Complete(r)
}

func (r *PodCounter) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
  podList := &corev1.PodList{}
  if err := r.List(ctx, podList); err != nil {
    return ctrl.Result{}, err
  }
  log.Printf("Number of pods: %d", len(podList.Items))
  return ctrl.Result{}, nil
}
```

there are way too many information hidden in here and for us to distill this completely, lets understand each and everything in a bit more details.

#### config

It's a `*rest.Config` from `k8s.io/client-go/rest`. It holds everything that is needed to talk to the kube api-server.

* the server url
* credentials (token, client certs)
* ca bundle
* qps
* burst limits, etc

This part gets the config in the following order

* `--kubeconfig` flag
* `KUBECONFIG` env var
* in-cluster config (via the service account mounted in a pod)
* `~/.kube/config`

#### `client.Client`

The `PodCounter{}` struct is just a holder which if needs to be a reconciler has to implement the `Reconcile(ctx, Request)` as part of the interface contract.

Embedding `client.Client` into the struct is an approach where we inject a reference to the client of the manager that we create as part `ctrl.NewManager`. 

#### things `NewManager` creates

`NewManager` builds a cluster object that wires several pieces from the rest.Config

* An HTTP client: which owns the actual connections (to talk to API server)
* A REST Mapper: which maps a Go type or GVK to a URL path
* A cache: registry of informers
* A client: which reads from cache and writes through REST.
* An API reader: an uncached client for when you need fresh reads from API-server.

#### What is an Informer?

Informer is a local cache kept in sync with the kube api-server, plus an event notification system on top.

It has these following components

* **Reflector**: talks to the API server
* **Delta FIFO**: is a queue of changes from the Reflector
* **Indexer**: is the in-memory store, a map keyed by `namespace/name` with optional secondary indexes.
* **Handlers** are callback that turn events into reconcile Requests.

![](/images/informer_list_watch_flow.png)

#### How does `SetupWithManager` works?
```
func (r *PodCounter) SetupWithManager(mgr ctrl.Manager) error {
  return ctrl.NewControllerManagedBy(&mgr).
           For(&corev1.Pod{}).
           Complete(r)
}
```
- The `NewControllerManagedBy` takes in the manager as the manager controlls the queue, cache and the client
- The `For` establishes the primary resource of the controller on which it reconciles. It's only a **type marker** (the builder looks it up in the scheme)
  - The informer is created for Pods (when this gets executed)
  - Every Pod event is turned into a Request{namespace, name} of that pod and put on in the work queue.
- The `Complete` finishes the chain and maps it to the reconciler `r` and does the real work as last step of the builder pattern.
  - build the controller with the work queue.
  - stores `r` as the thing to call when the request is taken off the queue.
  - registers the controller with the manager.

#### How we can watch multiple things?
