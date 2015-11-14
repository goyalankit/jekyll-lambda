---
layout: post
title: "cache basics"
date: 2014-09-12 11:40:19 -0500
comments: true
categories: 
---

Performance of CPU increase very fast, however memory is not involving as fast (not the latency nor bandwidth of the bus).

So we put in cache closer to the processor. Caches are small.
* Caches are faster because they are smaller.
* Speed of light limits the speed for larger memories.

### Locality
Programs tend to use data and insturctions with addresses near or equal to those recently used.

#### Temporal Locality
* Recently referenced items are likely to be referenced again in the near future.
* Replacement policy of cache.

#### Spatial Locality
* Items with nearby addresses tend to be referenced close together in time.
* Bring blocks of memory to the cache


```
sum = 0
for (i = 0; i < n; i++) {
  sum += a[i];
}
return sum;
```

In the above program:

Date:

* **Spatial Locality**: array accessed with stride-1 pattern
* **Temporal Locality**: sum referenced in each iteration

Instructions:

* **Temporal**: cycle through loop iteratively.
* **Spatial**: reference instructions in sequence.


#### Cache structure of core i7

<img src="{{root_url}}/images/post-images/core-i7.png"/>


### Cache Line or Block Size

In a direct mapped cache, cache line is the number of bytes that you can store at a given index (can be extended to set associative)

<img src="{{root_url}}/images/post-images/cache-line.png"/>


### Cache conflicts

#### Reads

1. **Cold(compulsory misses)**: Occurs on first block of data.
2. **Conflict Miss**: Occurs when cache is large enough but multiple data objects all map to the same slot. e.g. referring block 0, 8, 0, 8.
3. **Capacity Miss**: Occurs when the set of active cache blocks is larger than the cache.

#### Writes

1. Data may be stored at multiple levels of memory. (copies are spread across levels)
2. On **write-hit** (write block is in cache)
    * **Write-through**: Write immediately to memory
    * **Write-back** (defer write to memory until the line is evicted): Set a dirty bit to indicate if line is different from memory or not.
3. On **write-miss**:
    * **write-allocate**: load into cache, update line in cache. Good if more write to location follows.
    * **no-write-allocate**: just write immediately to memory.
4. Typical Caches:
    * write-back + write-allocate
    * wite-through + no-write-allocate, occasionally in case of multiple processors.
