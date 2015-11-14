---
layout: post
title: "Concurrent Objects"
date: 2014-11-17 18:40:16 -0600
comments: true
categories: os
---

Method calls take time and method calls by a single thread are always sequential.

### Compositional
A correctness property P is compositional if, whenever each object in system satisfies P, the system as a whole satisifies P. Basically we can compose objects satisfying a property P to create a more complex system that also satisfies P. For example, Quiescence.

---

### Quiescent Consistency

Principles:

1. Method calls should appear in one-in-a-time sequential order. --| **trivial** |
2. Method calls separated by a period of quiescence should appear to take effect in real-time order.

<!-- more -->
```
# not quiescent

Thread A --------------------|    r.write(7)    |---------------------------------------->

Thread B ---------------------- |  r.write(-3)    |-----------| r.read(-7) |------------->

---------------------------------------------------- ******* -----------------------------

****** period of quiescence
```

1. Property one ensures that we read a valid value. However one can read the old value always.
2. So in the above example, we should either read 7 or 3 since there's a period of quiescence


Informally,

- it says that any time an object becomes quiescent, then the execution so far is equivalent to some sequential execution of the completed calls.
- all operations appear to occur in some sequential order
- non-overlapping operations appear to occur in real-time order.
- Program order may not be preserved. I enque x and then y; deque operation overlaps both enqueues, you can come up with y.

```
# program order not preserved. Since no period of quiescence. But this is Quiescent Consistent.

Thread A --------------------|    r.enq(7)  |-------|  r.enq(-3)----------------------->

Thread B ------------------------------|   r.deq(-3)    |------------------------------>
```

#### Properties

- non blocking
- compositional
- doesn't respect the program order
- Quiescenly consistent with time
- Cannot touch multiple objects.
- equivalent to sequential order

---

### Sequential Consistency

Principles:

1. Method calls should appear to take effect in program order.
2. Method calls should appear in one-in-a-time sequential order. --| **trivial** |

- consistent with program order
- meet the object's sequential specification

---

Linearizability and sequential consistency
---

Linearizability: methods all at-a-time
