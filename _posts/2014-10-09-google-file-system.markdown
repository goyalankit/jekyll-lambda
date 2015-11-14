---
layout: post
title: "Google File System"
date: 2014-10-09 12:37:40 -0500
comments: true
categories: filesystem GFS OS
---

Source: http://static.googleusercontent.com/media/research.google.com/en/us/archive/gfs-sosp2003.pdf <br/>
Published: 2003

---

Design Constraints
---

1. Component failures are the norm rather than exception. FS consists of thousands if commodity parts and is accessed by comparable number of clients.
2. Files are huge by traditional standards. Multi-GB files are common. Billions of approximately KB-sized files.
3. Most files are mutated by appending new data rather than overwriting existing data.
4. Co-designing the applications and the file system API benifits the overall system by increasing flexibility.

Architecture
---

GFS consists of a **single master** and multiple **chunkservers** and is accessed by multiple clients.


Class Notes:
---

Problem in datacenters: failures.

**Master**: Single point of failure, master has log of operations.

Lease: soft state.

Revoke:

1. wait
2. contact server

---

Snapshot:

1. revoke lease
2. copy-on-write

snapshot on large files:-> copy on writes.

Contact replicas and tell them to make local copies. Update the metadata to point to the individual chunks.

- distributed the consistency to both application libraries.
- unique identifier could be a problem.

chunks metadata at master are not persisted.


Appends are ordered:

1) lease and primary
2) primary serial #'s

Master doesn't know about the namespaces either, it is also stored on chunkservers.

- No hardlinks/softlinks
- absolute paths
- no datastructure representing directories like inodes
- no datastructure to enumerate contents of one directory
