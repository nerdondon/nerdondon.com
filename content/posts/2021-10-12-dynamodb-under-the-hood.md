---
title:
  "[Notes]: AWS re:Invent 2018: Amazon DynamoDB Under the Hood: How We Built a Hyper-Scale Database
  (DAT321)"
date: 2021-10-12T00:00:00-08:00
description:
  "Notes for AWS re:Invent 2018: Amazon DynamoDB Under the Hood: How We Built a Hyper-Scale Database
  (DAT321)"
tags: [notes, database, distributed, dynamodb, reinvent]
logoText: "aws dynamodb delete-table --table-name notes"
---

## Prelude

Some notes from the **Amazon DynamoDB Under the Hood: How We Built a Hyper-Scale Database (DAT321)**
talk by Jaso Sorenson. The talk was given at AWS re:Invent 2018. Watch on YouTube
[here](https://www.youtube.com/watch?v=yvBR71D0nAQ).

## Notes

### `GetItem` and `PutItem`

- `GetItem` and `PutItem` initially go through a service called the request router that will then
  redirect the request to the appropriate storage node

- Data from a `PutItem` will be sent to the leader of a Paxos group formed by 3 storage nodes

  - Hearbeats are 1.5 sec and missing 2-3 (?) heartbeats will trigger an election

  - The storage nodes in a group are in different availability zones (AZ's)

- Request routers and storage nodes are zonal services (service that runs in multiple availability
  zones)

  - Request router is a stateless service

- A table is partitioned by the provided primary key and each partition is backed by a Paxos group
  of storage nodes

- For an eventually consistent read, the request is sent to a random storage node in its group

- Storage nodes are composed by 2 primary components: a B-tree and a replication log

### System management and Auto-admin

- Auto-admin is a process that serves many roles for system management

- It keeps the Partition Metadata System up to date with information like the current leader of a
  group of storage nodes

- It also handles partition repairs. If a partition node goes down, it clones a live node and brings
  the node up to date by replaying the replication log

### Secondary Index

- The table is partitioned by the secondary index and the partitions are each backed by a Paxos
  group of storage nodes. This is done completely independently from the base table.

- Changes to the replication log of the base table are propagated by a separate process called the
  Log Propagator

- Writes can cause an entire secondary index to be re-written. Big write amplification.

### Provisioning and Token Bucket Algorithm

- Token bucket algorithm used to measure capacity being used.

- The bucket will initially be filled with tokens to represent the requested DDB capacity (i.e.
  RCU's and WCU's). Each operation will remove one token from the bucket. The bucket will get
  refilled at a rate of 100 tokens/second.

- A request will be throttled if the bucket is exhausted.

- A token bucket will fill to a max capacity that is a multiple of your provisioned capapcity. This
  allows for bursty traffic.

### Global Tables

- Global tables can be thought of as an external service on top of DDB. It still has to go through a
  request router to replicate data from one region to the other.

- There is a stream reader (called `RepOut`) that consumes changes from DDB streams of the "child"
  (my word) tables that compose a DDB global table.

- There is a challenge that, when there is a partition split, you need a `RepOut` process reading
  from the DDB stream shard of the new partition.

- A `ReplAdmin` process will watch metadata about partitions from the DDB control plane and start a
  workflow that will tell a process in the `RepOut` process pool to begin processing from the new
  partition.

- `RepOut` will send batched replication messages to a destination regions `RepIn` service that will
  drive replication through request routers at the destination region. Once this replication is
  done, `RepIn` will notify `RepOut` of success and the stream will be checkpointed.

### TODO

- Maybe rest of talk about DDB specific backup mechanisms. The parts with notes are more generic and
  more interesting especially due to my current side-project developing RainDB.

## Useful references

- [AWS re:Invent 2018: Amazon DynamoDB Under the Hood: How We Built a Hyper-Scale Database (DAT321)](https://www.youtube.com/watch?v=yvBR71D0nAQ)
