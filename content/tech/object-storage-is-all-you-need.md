---
title: "Object Storage is All You Need: Justin Cormack"
date: "2024-12-01"
tags: ["object-storage", "s3", "distributed-systems", "kubernetes", "backend"]
description: "A gist of Justin Cormack's famous KubeCon talk — the CTO of Docker on why object storage is the foundational primitive of modern infrastructure."
---

A summary of one of the standout talks from KubeCon by Justin Cormack, CTO of Docker: *"Object Storage is All You Need."*

Amazon S3 has been an excellent primitive on the object storage front. As Jeff Bezos put it, it truly is the "malloc for the internet." But what exactly is an object store?

## What is an Object Store?

- A key-value (KV) store powered by HTTP APIs
- Allows listing functionality to some extent
- Objects can only be updated as a whole, not in parts
- Essentially a minimal, non-POSIX filesystem

The "non-POSIX" part is particularly important — let's dig into why.

## POSIX — Portable Operating System Interface

At a high level, POSIX is a set of standards to ensure that applications developed on one UNIX flavor can run on other UNIXes. (This compatibility is one of the reasons Linux remains relevant today.)

The POSIX standard describes how system calls must behave, with specific conditions that must be enforced:

- **Strong consistency** — if a write happens before a read, the read must return the written data
- **Atomic writes** — reads should return either all data written by a concurrent write, or none — never partial
- **Support for** random reads, writes, truncate, and fsync
- **File access control** — permissions and system calls like `chmod` and `chown`

## Why Isn't S3 POSIX-Compliant?

- **Atomicity** — PUT, GET, and DELETE are object-level and lack strict atomic guarantees
- **File locking** — POSIX requires locking mechanisms; S3 doesn't support this natively — it's designed for high-throughput, stateless operations
- **Consistency** — S3 provides eventual consistency in some scenarios, though it does offer strong read-after-write consistency for new objects
- **Hierarchical structure** — S3 only emulates directory structures using prefixes and delimiters (`/`), rather than true hierarchies
- **File modification** — POSIX allows modifying parts of a file; S3 requires the entire object to be overwritten

## How Concurrency in Object Stores Evolved

### Phase 1: The Beginning

S3's design was initially influenced by Amazon's experience shipping websites. With only one deployment pipeline, concurrency wasn't a primary concern.

### Phase 2: Content Addressing

Content addressing identifies and retrieves data using a hash of its content rather than its location or name. On AWS, you can generate signed URLs to write files with specific `sha256` hashes to a designated location. This offered some concurrency advantages but didn't solve everything.

### Phase 3: Database Integration

Databases were brought in to handle concurrency more robustly:

- Versioning columns for updates
- TTL keys for locking
- Row-based locking for specific records

### Phase 4: PUT with `If-None-Match: *`

A game-changer for distributed systems. This ensures objects are only written if they don't already exist — opening up new dimensions for managing concurrency in object stores.

## Where Things Stand Today

A lot has been built on top of these primitives:

- **Delta Lake** — high-performance ACID tables over object storage
- **Serverless ACID databases** — replicated with similar principles (e.g., Phil Eaton's work)
- **Log-based approaches** — Jay Kreps, founder of Confluent, has compelling insights here
- **Analytics databases** — Snowflake and Databricks are built on object stores, using open formats (loosely, Parquet files) as immutable logs for point-in-time queries

Other notable innovations:

- **Leader election via S3 conditional writes** (Gunnar Morling)
- **SlateDB** — an embedded database using an object store instead of local disk
- **TurboPuffer** — a local SSD cache in front of S3
- **Warpstream** — acquired by Confluent for over $100M, delivering Apache Kafka over S3
- **Bitdrift** — stores cold data on S3 with local storage and RAM for caching

This space is just getting started.

## References

- [KubeCon NA 2024 — Justin Cormack's Talk](https://kccncna2024.sched.com/event/984e605cee05b6b8afe6cd2c6c445127)
- [POSIX Filesystem Explained — Quobyte](https://www.quobyte.com/storage-explained/posix-filesystem/)
