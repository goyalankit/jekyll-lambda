---
layout: post
title: "cgroup basics"
date: 2014-09-15 19:08:43 -0500
comments: true
categories: linux cgroups
---

Source: https://www.kernel.org/doc/Documentation/cgroups/cgroups.txt

CGROUPS
===

Control **Groups** provide a mechanism for aggregation/participating sets of tasks, and all their future children, into hierarchical groups with specialized behaviour.

---

Definitions
===

### cgroup
A **cgroup** associates a set of tasks with a set of parameters for one or more subsystems.

Term              | Example
------------------|---------
Set of Tasks      | process ids
Set of parameters | 1 or 2 CPUs
Subsystem         | CPUSET, MEMORY % or nodes.

---

### subsystem

A **subsystem** is a module that makes use of the task grouping facilities provided by cgroup to treat group of tasks in particular ways.

- c`**group**` provides the grouping of tasks with a set of parameters.

A subsystem is typically a `"resource controller"` that schedules a resource or applies per-cgroup limits. But it may be anything that wants to act on a group of processes e.g. a virtualization subsystem.

### Hierarchy

A **hierarchy** is a set of cgroups arranged in a tree, such that every task in the system is in exactly one of the cgroups in the hierarchy, and a set of subsystems; each sub-system has system specific state attached to each cgroup in the hierarchy. Each hierarchy has an instance of the cgroup virtual filesystem associated with it.



Rule 1. A single hierarchy can have one or more subsystems attached to it.

Example: You create a single hierarchy for memory and cpu. It's good if you have one to one mapping. You can have:

1. `/cg1`: 40% memory, 60% CPU
2. `/cg2`: 60% memory, 40% CPU

{% img https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule1.png 500 %}

Rule 2: Any single subsystem (such as cpu) subsystem cannot be attached to more than one hierarchy if one of those hierarchies has a different subsystem attached to it already.


{% img https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule2.png 500 %}

Rule 3: A task can belong to only one cgroup in a hierarchy. All tasks on a system by default are member of default cgroup of that hierarchy, which is known as root cgroup.

{% img https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule3.png 500 %}

Rule 4: Any process that forks itself creates a child task. By default child task inherits the cgroup membership of its parent but can be moved to different cgroups as needed.

{% img https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule4.png 500 %}

source: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-Relationships_Between_Subsystems_Hierarchies_Control_Groups_and_Tasks.html
