---
layout: post
title: "Memory Resource Management in VMWare ESX Server"
date: 2014-10-02 22:35:46 -0500
comments: true
categories: 
---
Source: https://www.usenix.org/legacy/event/osdi02/tech/waldspurger/waldspurger.pdf<br/>
Published: 2002

VMWare ESX Server is a thin layer designed to multiplex hardware resources efficiently among virtual machines running unmodified commodity operating systems.

### Memory Virtualization

All guest operating systems expect a zero-phased physical address space as provided by real hardwares. ESX Server gives this illusion by adding an extra level of translation. In this mechanism, Virtual addresses are mapped to "Physical Address" (provided by ESX Server) which are further mapped to Machine addresses (actual memory on chip). Seperate shadow pages tables, which contains virtual to machine page mappings for use by the processor.

```
 -----------------      PMAP in ESX    ------------------     Page Table in OS   -----------------
| Machine Address |   <-------------  | Physical Address |  <-----------------  | Virtual Address |
 -----------------                     ------------------                        -----------------
```

#### Reclamation Mechanisms

ESX Server usually overcommits memory and needs a reclamation mechanism to reclaim memory from one or more virtual machines.

#### Balooning

A small balloon module is loaded into the guest OS as a pseudo-device driver or kernel service. It doesn't have any external interface. When ESX server wants to reclaim memory, it instructs the driver to 'inflate' by allocating pinned physical pages within the VM. This creates a memory pressure inside the VM and guest OS starts reclaiming memory to satisfy drivers request. The balloon driver communicates the physical page number to ESX server, which may then recalim the corressponding machine page.

* Technically guest os shouldn't touch pages allocated to balloon driver, however ESX server doesn't rely on guest os correctness. It annotates its pmap entry and deallocates its associated MPN (machine page number). any subsequent attempt to access that memory will generate a page fault that is handled by the server.

#### Demand Paging
ESX server preferentially uses ballooning to reclaim memory, treating as a common-case optimization if ballooning is not possible or insufficient, memory is reclaimed by paging out to an ESX Server swap area on disk, without any guest involvement.

A randomized page replacement policy is used so that higher level page semantics are not effected.

#### Sharing Memory - Content-Based page sharing

ESX server maintains a hashmap of pages where key is hash summarizing the content in page. If the hash value for new page matches an existing page, then the whole contents of the page are compared and if they match a reference count is incremented and the page is shared. Any attempt to write to a shared page will generate a fault, transparently creating a new page. (COW - copy on write)

If no match is found the page is hashed but it is not marked as COW. Instead it is tagged as a special hint entry. On any future match, contents of the hint page are rehashed and checked to see if the page has been modified. If the hash is still valid, a full comparison is performed and page is shared (marked as COW now).

#### Share based allocation

Resource rights are encapsulated by shares, which are owned by clients that consume resources. Traditionally VMs that have more shares are allocated more resources nad in case of reclamation, the VMs with lower shares are used to reclaim memory. It may happen that VMs with larger share are not using the memory and smaller VMs are the one that can actually benifit from the memory. So we don't have good resource utilization here.

ESX server introduces the concept of idle memory tax which charges more for the idle pages than the pages that are activally used.

#### Measuring Idle memory

ESX server uses a statistical sampling approach to obtain aggregate VM working set estimates directly, without any guest involvement. At the start of sampling period, a small number of the vm's "physical" pages are selected and their corresponding TLB enteries and MMUU state is invalidated. When VM access those pages, it leads to a fault and a use of the page is registered. At the end of period fraction f of memory actively accessed by the vm can be calculated by simply f = t/n (t page faults out of n invalidations)



