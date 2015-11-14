---
layout: post
title: "Exokernel: an OS architecture for application-level resource management - Notes"
date: 2014-09-07 15:34:49 -0500
comments: true
categories: os paper
---

**source**: http://research.cs.wisc.edu/areas/os/Qual/papers/exokernel.pdf<br/>
**published**: SIGOPS 1995

Traditionally operating systems define an interface between physical resources and applications. OS provides several abstractions like processes, files, address spaces and interprocess communication.

**These hardcoded abstractions limits**:

* Domain specific optimizations
* Discourages chages to the implementatinos of current abstractions
* makes it difficult or impossible the applications to define new abstractions

In exokernel these problems are solved by using **application level (i.e., untrused) resource management**. In exokernels, virtual memory and interprocess communication are implemented at application level. A minimal OS - *exokernel* basically multiplexes available hardware resource. These application level implementations are provided as libraries which application developers can directly or implement their own. Exokernel tries to implement a very low level interface, thus leaving a lattitude for more domain specific optimizations at the higher (application) level.

Exokernel rather than emulating hardware (virtual machines, high overhead), *exports* hardware resources. It employs three techniques to export resources securly:

1. **secure bindings**: applications can securely bind to machine resources and handle events.
2. **visible revocation**: applications participate in a resource revocation protocol
3. **abort protocol**: an exokernel can break secure bindings of uncooperative applications by force.

**use cases of exokernels**:

1. Page tables are application dependent and using domain specific implementation each application may implement their own page tables in exokernel. For example in database implementation.
2. The same is true for cache replacement policy, a paper shows that application runtime can be improved by 45% by implementing application specific caching policies.
3. End-to-end arguments applies to exokernels and new tecchnology can be more easily adapted by providing library operating systems (application level resource management libraries).

{% img http://i.imgur.com/wv54Rtd.png %}

### Library Operating Systems

1. Library operating systems need not multiplex a resource so they can be easuly specialized without consideration of resource management. 
2. Since lib OSs are treated as untrusted applications by exokernel , lib OS can trust the applications (which makes them easy to design). In case of invalid params to lib OS, exokernel will check the input (since untrusted) and declare it as invalid. No other applications will be effected. 
3. Finally there will be less kernel crossings since most the OS runs in address space of applications.

### Exokernel Design

> The challenge of exokernel is to provide maximum freedom to applications in managing physical resource while protecting them form each other. Exokernel separates protection from management through low level interface



### Exokernel Notes

Deadlock conditions:
* mutual exclusion
* non-preemptive
* ---
* ---

Thrashing vs Livelock

End-to-end argument: putting reliability in your system, when you have end-to-end reliability checking can only be viewed as optimization.









































