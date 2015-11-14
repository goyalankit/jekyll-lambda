---
layout: post
title: "Precise Miss Analysis for Program Transformation with Caches of Arbitrary Associativity"
date: 2014-09-12 12:01:54 -0500
comments: true
categories: 
---

**Source**: http://mrmgroup.cs.princeton.edu/papers/ghosh_asplos98.pdf <br/>
**Published**: 1998

Currently the gap between memory and processor is addressed by compile- and programmer- applied optimizations like matrix blocking, data structure padding and other program transformations.

This paper defined a CME (cache miss equation framework) which expresses memory references and cache conflict behavious in terms of set of equations.

Two main approaches to automatically improve data locality for loop-oriented programs:

1. **Loop nest restructuring** - *includes* permutation, tiling (parition loop space into chunks), fusion are used to reorder access patterns. These transformation have usually considered capacity misses in the past, however there could be conflict misses which are difficult to identify and are highly sensitive to problem size and base addresses.
2. **data layout optimizations** - padding to the data structures.

---

CMEs are a system of **linear Diophantine equations**

Assumptions:

* Loops are assumed to be normalized, such that step value is 1.
* Consider only perfectly nested loop and some imperfectly nested loop if they have one block between the nests.
* Loops contain no conditional statements.
* All load/store inside loop nest belong only to array.

### Cache Miss Equations

#### Cold Cache Misses

1. Either the first access along the direction of vector.
2. accesses that have just crossed a memory line boundary along the direction of vector.

#### Replacement Miss Equations

1. Example: for an access stream - Ra, Rb, Ra. A conflict clearly occurs in direct mapped cache if Ra and Rb maps to same cache line.

This happens:
```
Memory_Address_of_Ra = Memory_Address_of_Rb + n * Cache Size + Line_Size_Range
```

Roughly a conflict occurs if memory addresses accessed differs by multiples of cache size.

In case of k-way set associative cache, a miss occurs if between consecutive access to a memory line, at least k other accesses occur to distinct memory lines that map to the same cache set.


### Algorithm

1. Find all reuse vectors along an iteration point.
2. Order all reuse vectors in lexicographical order.
2. Find potential conflict points: p = i - r
3. Solve CME equations for each reuse vector and the solution at a point could belong to cold miss or conflict miss.
4. If the solution satisfies cold miss equations, mark them as "intermediate points" otherwise it is a conflict miss.

**Set-Associative Cache**

Each CME soluction can be summarized using a triple (i,j,n). n denotes the cache "wrap-arounds", and same value of n indicates the conflict for same memory line.

For k-way associative cache, maintain a set of disctinct conflict points with same n and if the set size exceeds k, we have a conflict.

### Solutions to CMEs

#### Padding

Find constraints on CMEs using cache base address, dimensions size so that no solution exists for CME. Decide padding based on those constraints.
