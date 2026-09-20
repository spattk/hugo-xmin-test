---
title: Kubernetes Service Account Token and Lifecycle
date: 2026-09-20T08:15:00.000-07:00
author: Sitesh Pattanaik
---
### Phases

* **Phase 1: Pod Scheduling and Assignment** 
* **Phase 2: Projected Service Account Token Provisioning** 
* **Phase 3: The Secure TLS Handshake & API Server Spoof Prevention**

* **Phase 4: API Authentication (AuthN) and Authorization (AuthZ)** 



#### Phase 1
- A pod is deployed via applying the `kubectl apply -f pod.yaml`
- As kubectl is getting the request, it persists the required object in ETCD.
- The scheduler sees that there is a new pod that is not bound to a node.
  - it figures out the right node for this pod based on all the constraints as well as capacity on the node
  - it updates the pod's `nodeName` based on that.
- The kubelet on the NodeName which is constantly watching the api-server for it's node name, notices the assignment

#### Phase 2
- The kubelet starts preparing for the pod.
   - The kubelet realizes it would require a ServiceAccount for the pod first.
   - calls the api-server via the `TokenRequest API` endpoint
- API-server creates a JSON payload containing (service account name, pod UID, namespace, expiration), signs it with its own private key and this forms the JWT token.
- The API-server hands back this token to kubelet.
- The kubelet, creates a folder in the host machine and stores these 3 things
  - token (the one sent above)
  - ca.crt (the cluster's root CA public cert - which it copies from host files)
  - namespace
- the kubelet binds this to `/var/run/secret/kubernetes.io/serviceaccount`


#### Phase 3
- the application inside the pod wants to talk to the api-server
  - it reads the `token` and `ca.crt` from it's local path
- the pod initiates the HTTPS TLS request
  - the api-server responds by sending TLS Server Cert
  - the pod validates if this cert is signed by the same ca.crt i have on my disk



#### Phase 4
- The pod sends the token in the HTTP request as part of Authorization: Bearer <token> 
- The API-server reads it and validates using it's public cert and confirms the identity of the service account.
- Then the request is handed over to the RBAC engine for next steps.
