---
title: "The \"apply\" Evolution in Kubernetes"
date: "2026-03-07"
tags: ["kubernetes", "devops", "kubectl", "cloud"]
---

In Kubernetes, `kubectl apply` is the bridge between **desired state** (the YAML file) and **live state** (the cluster). The "apply" logic determines how the cluster calculates the difference between these two states and who is allowed to change what.

## Client-Side Apply (The Legacy Method)

- **Logic:** "math" happens on the client side — your local machine
- **Mechanism:** uses a `last-applied-configuration` annotation stored on the object
- **3-way merge:** kubectl compares the local file, the live state, and the last applied annotation
- **Flaw:** It's a "last-writer wins" system — blindly applies any change proposed at the last minute as the final state
- **Win:** Simple and compatible with very old versions of Kubernetes where field ownership doesn't exist

## Server-Side Apply (The Modern Standard)

- **Logic:** "math" moves to the Kubernetes API server
- **Mechanism:** uses `managedFields` in the object metadata
- **Field ownership:** SSA introduces the concept of a *field manager* — every field in a manifest is "owned" by the person or tool that applied it
- **Conflict detection:** if two managers try to control the same field with different values, the API server blocks the change and throws a conflict error
- **Win:** enables cooperative management — HPA can own the number of replicas while a human operator owns the image version

## Technical Deep Dive

The secret to SSA is the `managedFields` ledger. You can inspect it by running:
```bash
k get <resource> -o yaml
```

Key concepts in the ledger:

- **`f` (fields)** — represents specified keys owned by a manager
- **`k` (keys)** — represents specific items in a list being tracked
- **Field manager name** — set to `kubectl` by default, but customizable via `--field-manager` flag to identify different CI/CD pipelines

## Conclusion

As Kubernetes matures, SSA is the clear winner for production environments. It transforms the API server into a smart traffic controller that prevents state flapping when multiple managers own different parts of the same manifest.
