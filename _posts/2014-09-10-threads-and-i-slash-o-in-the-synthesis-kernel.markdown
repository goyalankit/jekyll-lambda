---
layout: post
title: "Threads and Input/Output in the Synthesis Kernel"
date: 2014-09-10 22:22:42 -0500
comments: true
categories: os paper
---

Source: http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.51.3887
Published: 1989

Synthesis operating system kernel combines several techniques to provide high performance, including kernel code synthesis, fine grain scheduling and optimistic synchronization.

Goals:
1. High performance
2. self-tuning capability to dynamic load and configuration changes
3. a simple, uniform and intuitive model of computation with a high-level interface

Two kind of objects supported by synthesis kernel: Threads and I/O


### Synthesis model of computation

To support parallel and distributed computing, the threads of execution form a directed graph, in which the nodes are threads and the arcs are data flow channels.

Synthesis threads are threads of execution. Some threads never execute user-level code, but run entirely within kernel to provide additional concurrency for some kernel operations. Threads execute program in quaspace (quasi address space) which also stores data. Finally, I/O devices move data between threads including files and messages.

#### No virtual memory in current implementation

On one physical node, all the synthesis quaspaces are subspaces of one single address space. Kernel blanks out parts of address space that each quaspace is not supposed to see. Since they are part of same quaspace, it is easy to share memory by setting their address mappings.

## Kernel Code Synthesis

1. Factoring invariants: bypasses redundant computations.
2. The Collapsing layer method eliminates unnecessary procedure calls and context switches, both vertically for layered modules and horizontally for pipelined threads.
3. The executable data structure method shorten the data structure traversal time when DS is travelled always the same way.

Synthesis kernel can be devided into a number of (collections of procedures and data). These collection of procedures are called - quajects that encapsulate hardware resources (provide interface to hardware through quajects). Important quajects include threads and I/O devices servers.

Threads (quajects, that encapsulate hardware) are abstraction of CPU. The device servers are abstraction of I/O devices. Except from threads, quajects consist only of procedures and data.

quajects don't support inheritance or any other feature, events such as interrupts start the threads that animate the quajects and do work.

Building blocks of quajects:

1. monitors, queues and schedulers.
2. switches (equivalent to C switch statement, direct interrupts to the appropriate servie routines), pumps (contains thread that actively copies its input into its output.) and guages (counts events like procedure calls, data arrival and interrupts; used by scheduler).

pumps connect passive producers to passive consumers.

### Queues

Queue Type => Synchronous (blocks at empty or full) and Asynchronous(signals at those conditions) => dedicated (only 1 producer or consumer, omits synchronization) and optimistic queues (accepts queue insert and delete from multiple prod;cons;)


### Threads are created by **Quaject Creator** in 3 stages

1. **Allocation** - allocate memory - for quaject and procedures.
2. **Factorization** - uses factorizing invariants to substitute constants into quaject's code templates.
3. **Optimization** - do peephole optimizations.

```
# mnemonic
 -----\|\ -> [  - - \ - | \]  -> [ - - - c \ - c | ] -> [ - - - - c - - c ]
```

### Quaject Interfacer

Starts the execution of existing quajects by installing (load procedures and data) them in invoking thread. 4 Stages:

1. find connecting mechanism (queue, monitor, pump or a simple procedure call) from 1 quaject to another.
2. Factorization and Optimization (Run time).
3. Dynamic link stage stores synthesized code's entry points to the quajects.

### Reducing Synchronization

Three methods to reduce Synchronization:

1. Code Isolation (synchronization avoidance) -> kernel synthesis to separate independent modification of DS. e.g., Thread Table Entry (lock global table -> change local tables)
2. Procedure Chaining ( "" ) -> serializes the execution of conflicting threads. Signal during interrupt handling could be dangerous, so chain the prodedure invoked by signal to the end of interrupt handler (replace the return address - effecient.)
3. Optimistic Synchronization - update without synchronization, test at the end - if something bad happened -> rollback and retry.


### Optimistic Queues

Queues are important since lot of functionality is implemented using queues. Four types - SP-SC, SP-MC, MP-SC, MP-MC

* **SP-SC** - synchronization is only necessary when queue becomes empty or full. Usually busy wait.
Solution: update the pointer (Q-head for producer, Q-tail for consumer) at the very end. In later architectures, you may have to put memory fence to ensure store order.

* **MP-SC** - compare-and-swap instruction + retry loop.
Now producers claim the space first by incrementing the Qhead and if successful they start inserting the elements. Consumers can't trust the head now, so producers update flags corresponding to queue positions indicating if the element is valid.

Compare and swap is atomic, so they update a local pointer and see if it's valid. If it's valid then they swap it with head. -> all other producers will see the updated head now while the current producer is inserting elements on claimed space.

### Threads

The thread state is defined by TTE containing register save area, interrupt handlers, error traps and signal vectors; the address map tables, **context-switch-in and out procedures**.

Kernel generates some code for threads in a protected area - custom system calls, synthesized interrupt handlers such as for queue buffering, specialized eror trap handlers and thread calls (start, stop, etc.)

* Thread switches to supervisor mode while running kernel call instead of kernel server running it for the thread. Kernel quaspace is made accessible.

Waiting queue is spread accross resources and waiting thread's unblocking procedure is chained to the end of interrupts handler.


### Context Switch
1. Switch only the part of context being used. (most of the processes don't use FP co-processor so don't save state [default synthesized code] until used[then resynthsize code]).

2. use executable data structures to minimize critical path. - No "dispatcher" - context-switch-out of 1 process contains jmp to context-switch-in for next process.

### Signal
Signal is an asynchronous software interrupt sent by a thread to another. Signal system call modifies a general area so that receiving thread can run the signal handler code in used mode.

### Scheduling

Fine grained quanta to threads - based on "need to execute" determined by rate at with I/O data flows into and out of its quaspace.

Effective CPU = Quanta to thread / Quanta to all threads

priority -> assigning large quanta or placing in front of ready queue.

### I/O

At boot time, kernel creates the servers for raw physical devices. A simple example pipelines the raw input from tty to filters. multiple filters may be applied during the process.

### Producer/Consumer in I/O

Active producer - Passive Consumer (or vice versa). In case of SP-SC, a procedure call from consumer asking producer for data does the job.

For MP-SC or SP-MC, attach monitor at the multiple end resulting in MP-SC or SP-MC queues.

Passive producer - Passive Consumer -> xclock -> clock producer ready to produce time and clock displayer ready to display at all times. We use PUMP here that reads from one end and writes to another.

### Interrupt Handlers

Each threads has its own synthesized interrupt handellers and system calls. Lot of interrupts can be handelled in current execution thread only so minimal saving of registers is required and it avoid a context switch. During short level interrupt, higher level interrupts may happend and those inerrupts are chained.
