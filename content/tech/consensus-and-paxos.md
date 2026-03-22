---
title: "Consensus and Paxos"
date: "2026-03-22"
tags: ["distributed-systems", "consensus", "paxos", "backend"]
---

Majority agrees on a proposal → consensus. The reached consensus is eventually adopted by everyone.

## Why Do Systems Need Consensus?

Once you hit enough scale, vertical scaling stops being an option — you have to go horizontal. That introduces the problem of keeping multiple nodes aligned on the same state.

There are two common schemas this leads to:

**Leader-Replica** — one node leads, others replicate. If the leader becomes unavailable, the remaining nodes must elect a new one — that requires consensus.

**Peer-to-Peer** — every node is at the same level, with no designated leader. Nodes need to maintain consensus all the time.

## Paxos

### Roles

- **Proposer** — proposes a new value
- **Acceptor** — accepts the proposed value
- **Learner** — learns about the agreed value

A single Paxos node can take on multiple roles — even all three simultaneously.

### Key Properties

- Paxos nodes must be **persistent** — they cannot forget what happened
- Paxos aims at reaching a **single consensus**
- Once consensus is reached, it **cannot progress** to another consensus in the same run
- To reach a new consensus, a **new Paxos run** must be started

## Contention

Contention is a serious problem in Paxos and can effectively stall a run. Here's how it plays out:

1. Proposer-1 sends `prepare(5)` to two of three acceptors
2. Meanwhile, Proposer-2 sends `prepare(6)` to the same acceptors — before the `accept-request` from Proposer-1 arrives
3. Acceptors reject Proposer-1's `accept-request` since they've seen a higher proposal number
4. Proposer-1 retries with `prepare(7)` — now blocking Proposer-2's `accept-request`
5. This can loop indefinitely, stalling the entire run

### Fix

Introduce **exponential backoff** between retries so proposers don't keep knocking each other out in lockstep.

## Further Reading

- https://www.youtube.com/watch?v=d7nAGI_NZPk