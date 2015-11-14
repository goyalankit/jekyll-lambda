---
layout: post
title: "process address space"
date: 2014-10-15 18:40:53 -0500
comments: true
categories: os process notes
---

* On IA-32 systems with 2^32 = 4GiB, the address space is usually split in 3:1 ratio. The kernel is assied 1GiB, while 3GiB is available to each userspace process.
* The contents of kernel address space are always same regardless of which user process is active.


Layout of the Process Address Space
---

1. **text segment**: binary code of the code currently running.
2. code for dynamic libraries.
3. heap where global variables and dynamically generated data are stored.
4. stack used to store local variables and to implement function calls.
5. Sections with environment variables and command-line arguments.
6. Memory mappings that map the contents of files into the virtual address space.


Each process has `mm_struct` instance that can be accessed by task structure.
