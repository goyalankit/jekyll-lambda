---
layout: post
title: "Paxos Notes"
date: 2014-10-29 16:57:37 -0500
comments: true
categories: paxos "distributed systems"
---

Source: Paxos Made Practical - David Mazieres

**Paxos** is a simple protocol tat a group of machines in a distributed system can use to agree on a value proposed by a member of the group.

If it terminates, the protocol reaches consensus even if the network was inreliable  and mutiple machines simultaneously tried to propose different values.

The Basic idea is that each proposal has a unique number.

<!-- more -->

### State Machine Replication

