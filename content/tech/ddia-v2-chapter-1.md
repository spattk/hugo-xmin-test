---
title: "Chapter 1: Trade-offs in data systems architecture"
date: "2026-03-07"
tags: ["distributed-systems", "cloud", "data-intensive", "book-cloub"]
---

### Chapter 1: Trade-offs in data systems architecture

### intro

- data intensive if data management is one of the primary challenges
  - storing and processing large data volumnes
  - changes to data
  - ensuring consistency
  - highly available systems
- compute intensive for compute related


### frontend vs backend

- frontend needs to handle only user's data
- backend manages data behalf on all users
- backend service is reachable via HTTP or WebSocket
  - interacts with one or more databases
  - interacts with additional data systems called data infrastructure


### operational vs analytical systems

#### operational systems

- where data is created by serving external users
- both reads and modifies the data in db
- analytical systems
- contains read-only copy
- optimized for data processing and analytics
- became OLTP (online transaction processing)
  - because these applications are interactive == online
  - transaction has remained from the world of proceessing money


#### refer to the table for more information

- point queries vs aggregate queries
- crud vs bulk import
- external use-cases vs internal
- fixed small queries vs complex big queries
- latest data vs historical data in general
- gb vs tb
- a hybrid exists where product-analytics or real-time analytics
  - like pinot, druid, or clickhouse
  - ingest data in realtime and optimized for low-latency query responses


### data warehouse and data lake

#### data warehouse

- contains a readonly replica of the prod db for analysts
- analysis friendly, cleanup data
- it's derived from multiple source to enrich
- etl : extract, transform and load
- mostly a relational data model
- less suitable for feature engineering, nlp etc use cases


#### data lake

- solves the use case for data scientists
- simply contains files of data without imposing any file format
- avro or parquet
- cheaper than relational data stores
- etl processes -> data lake is intermediate stop
- note: both systems feed data from operational systems via ETL


### cloud vs self-hosting

- build or buy


#### pros

- easier maintenance
- good for spiky workloads
- better quality of service


#### cons

- less control on feature dev
- less control on recovery
- less control on pricing
- geopolitical conflicts
- hybrid approach has worked out for some


#### separation of storage and compute

- in traditional computing, disk storage == durable
- RAID (in h/w or in s/w) is used to tolerate single node failures
- in cloud, local disks attached -> they are ephemeral cache
- alternative to local disk == virtual disk storage (amazon ebs)
  - virtual disk susceptible to network failures
  - because every i/o operation is a network call
  - to avoid this and use dedicated storage service like s3
  - hence, cloud db typically manage smaller values in separate service and large in object stores
- in cloud systems, the storage and compute and usually segregated
- multi-tenant systems
  - requires careful system to avoid noisy neighbour


#### Note

- capacity planning == financial planning
- performance optimization == cost optimization


### distributed vs single-node

#### when to prefer distributed

- application is inherently distributed like communication between two devices
- requests between cloud service has to travel through network
- high availability
- scalability
- latency for global traffic
- easy scale up and down
- specialized hardware like gpu
- legal compliance like gdpr
- spiky workloads sustainability


#### problems with distributed systems

- slower network call
- more possibility of failures
- dealing with large data: easier to bring computation to data not other way round
- cheaper


#### Note

- distributed systems are complicated. should be avoided until absolutely necessary


### microservices and serverless

#### advantages of microservices

- reduced cross team coordination
- cleaner interface for plug and play
- team free to change implementation
- no shared data


#### disadvantages

- complexity and unnecessary overhead
- redundant infrastructure


#### serverless

- used in different ways but it effectively uses servers underneath
- generally used to reflect elastic/auto-scaling functionality


### cloud computing vs super-computing

#### use of supercomputers over cloud-computers

- highly computationally intensive operation
- runs big batch jobs with checkpoints
- communication through shared memory and RDMA (trusted users)
- specialized network topology
- assume all compute nodes are closeby and doesn't require network calls


### data systems, law and society

- law and regulations impact the data design
- gdpr, opt out systems impact append only logs that are efficient
- no standard/easier way
- things have pros and cons
- sometimes data minimization is helpful to not deal with all these things
- balance the needs of business with the needs of the customers