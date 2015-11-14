---
layout: post
title: "Spanner: Google's Globally Distributed Database"
date: 2014-11-06 18:47:50 -0600
comments: true
categories: os spanner distributed-database
---

**Spanner**: Database that shards data across many set of Paxos state machines in data centers spread accross the world.


### Time API

Sawtooth peak when executing, comes back when synced from server.

Smax: largest timestamp till relpica is up to date.

### Leases
leaders participate in local and distributed transactions. So it's better to have leaders for long time. Auto released after expiration.

extend the lease:
- leader can ssign timestamp to lease in future.
- leader can try to extend it's lease to make sure it's present when transaction happens.

<!-- more -->

### Snapshot isolation: all reads satisfied from the image.

- not as strong as strict serializability.
- supported using Tsafe -> time when you don't have prepare transactions.
