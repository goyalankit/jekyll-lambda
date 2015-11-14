---
layout: post
title: "file system continued"
date: 2014-09-09 15:56:30 -0500
comments: true
categories: 
---

{% img http://i.imgur.com/ZXnlN2R.jpg?1 500%}

**fsync**: transfers (flushes) all modified in-code data of the file to the disk drive. The call blocks until the device reports that the transfer has completed.

**fdatasync**: is similar to fsync but it doesn't flush modified metadata unless that metadata is needed in order to allow a subsequent data retrieval to be correctly handled

source: http://www.hackinglinuxexposed.com/articles/20030616.html
```
flock(fd, operation)
    Locks or unlocks an entire file. Doesn't work on NFS mounted filesystems, unfortunately.
    Available operations are

    LOCK_SH	Create a shared lock
    LOCK_EX	Create an exclusive lock
    LOCK_UN	Release our lock
    LOCK_NB	Don't block when setting the lock. Return an error if the action cannot be completed.

lockf(fd, operation, offset)
    Locks or unlocks the portion of the file after offset. Allows you to lock just trailing portions of the file,
    if desired. lockf is just a front end for fcntl. Arguments:

    F_LOCK	Create an exclusive lock
    F_TLOCK	Create an exclusive lock or return an error immediately
    F_ULOCK	Release our lock
    F_TEST	Test if the file is locked by another process.[3]

fcntl(fd, command, struct flock );
    fcntl offers you the most locking control. The flock structure details the beginning and end of the segment to lock.
    It has arguments analogous to those for lockf and can differentiate between read and write locks.
    (fcntl can do other things too, such as setting the close on exec flag, duplicating a file descriptor, and more.)
```



Virtual File System
---

### file lookup

{% img http://i.imgur.com/xzBvOct.png?1 500 %}<br/>
source: Profession Linux Kernel Architecture, chapter 8

Symbolic links are directory enteries which doesn't contain any data but just a pointer to filename. A separate inode is used for each symbolic link. The data segment of inode contains a character string that gives the name of the link target.

File descriptors with introduction of multiple namespaces and linux containers can no more uniquely identify a file. A unique representation is provided by a special data structure (struct file).
