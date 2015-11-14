---
layout: post
title: "scheduling basics"
date: 2014-09-20 16:49:25 -0500
comments: true
categories: 
---



Scheduling policy in Linux Kernel
---

source: [Understanding the Linux Kernel](http://idak.gop.edu.tr/esmeray/UnderStandingKernel.pdf), Chapter 7

CPU time is divided into slices, one for each runnable process. Time sharing relies on timer interrupts and is thus transparent to processes.


For scheduling processes are traditionally classified as: I/O-bound or CPU-bound.

Another classification distinguishes three kind of processes:

1. **Interactive Processes**: Constantly interact with users, these programs spend lot of time waiting for keypress and should be woekn up quickly when input is received. e.g., shell, text editors, etc.

2. **Batch Processes:** Not much user interaction and often run in background. Often penalized by the scheduler. e.g., compilers, database search engines and scientific computations.

3. **Real-time process:** Stringent scheduling requirements, should never be blocked due to low priority processes. e.g., video and sound applications, controllers collecting data from sensors.


Process Preemption:
---

Linux processes are preemptable (both in Kernel or User mode). When a process enters TASK\_RUNNING state, the kernel checks if its dynamic priority is greater than the priority of the currently running process. If it is, execution of `current` is interrupted and scheduler is invoked to select another process.

e.g., 2 programs - text editor and compiler. Compiler is running, user presses a key, an interrupt is raised and the kernel wakes up the editor process. Kernel also checks that the dynamic priority of text editor program is higher than compiler, so it sets `TIF_NEED_RESCHED` for the current process (thus forcing the scheduler when interrupt handler reutrns.). As the interrupt handler returns, scheduler is invoked and the context switch happens. Note: preempted process is not suspended, it remains in TASK\_RUNNING state.

Quantum size
---

If quantum is too short, context switches get expensive. If it's too long, it may make system unresponsive. e.g., two users launch process together, one of them is interactive and other is batch. If scheduler schedules batch first initially, other process becomes unresponsive.

*Note that too long a quantum doesn't degrades the response time of interative processes in general due to process preemption.*


The choice of quantum is usually a compromise and Linux chooses a duration as long as possible, while keep a good response time.
