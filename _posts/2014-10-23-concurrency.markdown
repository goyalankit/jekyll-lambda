---
layout: post
title: "concurrency"
date: 2014-10-23 12:38:21 -0500
comments: true
categories: os concurrency
---

* Determinism doesn't effectively solves the problem.
* Symbolic execution


## Transactions and Concurrency

### ACID

**Atomicity:** All or nothing. Log to clean if system fails. Output commit -> you are screwed.
**Consistency:** Internal consistency of the database.
**Isolation:** Executes if it were running along.
**Durability:** Results will not be lost.

Memory Transactions: no guarantee, what if power goes down.

<!-- more -->

---

Isolation: some serial execution this is equivalent to.
Serializability: One approach to isolation - not dominant. Linearizability is dominant.

Transactions: 2-phase commit, 2-phase locking.

Serializability allows you to opearte on multiple objects.

Serializability and Sequential consistency might not correspond to real time order.
database - serializability. memory system: read/write.

composable: consisten with real time and doesn't touch multiple objects.


Single write gets broken into multiple writes inside kernel. Single write is not serializable

http://www.cs.rochester.edu/courses/254/fall2013/notes/12-concurrency
