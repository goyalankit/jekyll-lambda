---
layout: post
title: "practical cgroups"
date: 2014-09-15 20:43:56 -0500
comments: true
categories: 
---

#### To list cgroups of a particular process (say PID= 16455)

```
$ cat /proc/16455/cgroup
```

Example Output:
```sh
# PID = 16455

goyal@fyra:/sys/fs/cgroup$ cat /proc/16455/cgroup

11:name=systemd:/user/1002.user/5.session
10:hugetlb:/user/1002.user/5.session
9:perf_event:/user/1002.user/5.session
8:blkio:/user/1002.user/5.session
7:freezer:/user/1002.user/5.session
6:devices:/user/1002.user/5.session
5:memory:/user/1002.user/5.session
4:cpuacct:/user/1002.user/5.session
3:cpu:/user/1002.user/5.session
2:cpuset:/
```

#### Mount existing cgroup hierarichies onto file system

```
mkdir /sys/fs/cgroup/cg1
```
