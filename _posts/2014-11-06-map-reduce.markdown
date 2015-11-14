---
layout: post
title: "map reduce"
date: 2014-11-06 12:31:15 -0600
comments: true
categories: map-reduce os
---


**map**: processes a key/value pair to generate a set of intermediate key/value pairs.

**reduce**: merges all intermediate values associated with same intermediate keys.

```
map    (k1, v1)       -> list(k2, v2)
reduce (k2, list(v2)) -> list(v2)
```

All the mappers have to finish before reduces can end.

<!-- more -->

{% img http://i.imgur.com/At7gcgY.png?1 %}

- Mappers based on input sets
- Reduces based on number of output files
- Potentially less robust, you lose type information from map to reduce. Possibility of garbage values.
- Mappers read from GFS but write to local file system. Since it's faster to write locally.
- Reducers again write to GFS.


### Criticism

- Step backwards in programming paradigm.
    - schemas are good.
- Consistency model. Garbage values can creep in.
- No query language.
- More on the database side of the map-reduce.
- No indexes are used.
- Skew is the real problem.
