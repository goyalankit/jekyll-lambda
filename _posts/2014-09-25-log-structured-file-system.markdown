---
layout: post
title: "log structured file system"
date: 2014-09-25 14:37:15 -0500
comments: true
categories: linux filesystem
---

In LFS, log is the only structure on disk. The log is divided into segments (to maintain space for large files) where data is written sequentially. **segment cleaner** compresses the information from heavily fragmented code.

* Log also stores indexing information so that the files can be read back with efficiency comparable to current file systems.

LFS is based on assumption that files are cached in main memory, so the majority of traffic write traffic which is efficient since we are writing sequentially and eliminating seeks in the disk.

- Sprite LFS uses 60-75% of the disk bandwith where as unix file system uses about 5-10% since most of the time is spent in seeking.

---

The main challenge of LFS is to ensure that there are always large extents of free space available for writing new data. Large extents called *segments* are used, where segment cleaner continually regenerates empty segments by compressing the live data from heavily fragmented segements.

> File system is governed by two basic building blocks: technology and workload.

Three components that are significant for file system design:

1. **Processors**- getting faster and fater, putting more pressure on memory
2. **Main Memory** - Can cache files and change the predominant type of data, like more writes than reads, so you can optimize you FS for writes
3. **Disks** - well that's where you store your data, they have two components:
    - Transfer Bandwith -  can be improved using disk arrays and parallel-head disks.
    - Access Time - depends on mechanical head (at the time of publishing).

---

* Unix file system writes metadata structures such as directories and inodes synchronously, which is not good for lot of small files.
* LFS buffers a sequence of file system changes in the file cache and writes them sequentailly in a single disk operation.

Key Issues
---

1. Information Retrieval  - uses inodes and maintains a inode map which contains the location of inodes.
2. Large free space


