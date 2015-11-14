---
layout: post
title: "xen - the art of virtualization"
date: 2014-09-23 13:03:46 -0500
comments: true
categories: xen
---

x86 virtualization challenges
---

* EFLAGS register has interrupt enable flag(IF). If the kernel is being virtualized it doesn't have privilage to enable interrupts. Kernel calls pushf and popf all over the place.
* Register cr3 points to the base of page table. Hardware MMU will read cr3 on page faults if the virtual addressing is enabled. VMM can't trust OS, solved by either shadow page tables (ESX Server) or allowing updates after verification (XEN)
* Untagged TLB means frequent flushes. Addressed with hardware translation.



### Consider page fault

ESX Server

```Ruby
Guest application: refers to unmapped memory address.
Hardware: TRAP! set privilaged bit. jump to VMM trap handler
VMM: Figure out which guest OS, clear privilaged bit, check shadow page table (or software TLB cache), if page not resident, set up trap registers for guest OS; jump to that guest OS's trap handler.
Guest OS: Read cr2 to find the faulting address
HW: TRAP! You can;t read the cr2; set privileged bit; jump to VMM trap handler.
VMM: Read the cr2 and put faulting address it into guest OS register; clear privileged bit.
Guest OS: Allocate physical page and write page table entry from VA -> PA
HW: TRAP! you can't write(READ-ONLY access) to page table. set privilaged bit; jump to VMM handler
VMM: Allocate a page of machine memory,record PA->MA mapping and install it in shadow page table; clear privileged bit; return to guest OS trap handler.
Guest OS: everything worked fine. `iret` Clear the privileged bit and return to guest handler
HW: TRAP! (you don't get to clear the priveleged bit); set privileged; jump to VMM handler
VMM: `iret` to guest application
```


### XEN

**For each privilage level, a separate stack is maintained.**

Guest OS runs in Ring 1, Application runs in Ring 4 and VMM runs in Ring 0.

When application is executing it has its own stack in Ring 4, now when we have a page fault, hardware traps and calls the trap handler. Trap handler is directly registered by Guest OS (though validated by XEN). Trap handeling code runs in Ring 1 (guest OS)


#### Memory:
* Guest OSes allocates and manage hardware page tables.
* Guest OS allocates and registers with xen giving away its write privileges.
* All updates (using hyper calls) need to be validated by XEN.
* XEN registers page tables directly with MMU.
* To aid validation frames are marked: page directory, page table, local descriptor table, global des. tab., Writable.

#### CPU:
* basic assumption: os is most privileged., rings help.
* Privileged instructions are paravirtualized by requiring them to be validated and executed within xen. Any guest os attempt to execute privileged instruction: failed by processor (silently or trap).

* Exceptions and trap are virtualized straightforwardly. A table describing the handler for each type is registered with XEN for validation. exception stack frames are unmodified in paravirt.
* Excepions can be directly handled by hardware, but page faults need to go through XEN.
* Page fault handler would read cr2(not possible), copy it to the extended stack frame.
* On page fault, xen copies stack frame to guest os and return control to registered handler.Trap handlers (guest OS) are in Ring 1. CR2 is in ring 0. M2P

#### Device I/O
* clean and simple abstractions.
* interrupts -> lightweight event delivery mechanism.

#### Control and Management:
* Hypervisor will be involved in scheduling but there is no need to be involved in high level details such as how to share the CPU among domains. or what kind of packets a domain can transfer.
* Hypervisor provides control operations through an interace accessible from authorized domains.
* Domain 0 is created at boot time and has access to control interface and is responsible for application level management software.
* Control interface provides the ability to create/destrot domains, their scheduling parameters, physical memory allocations and access they are given to physical resources like physical disk and network devices.
*

### Class Notes:

Domain 0 - is a VM. Device drivers are put inside domain 0.

write a block of data ==> goes to VMM ==> goes to VM (converting it to actual blocks).

---
Not XEN RELATED
Hardware virtualization
system calls and page faults don't trap.


VA - PT - PA

VMWare
VA - Shadow PT - MA
PA - PMAP - MA



Shadow tables - per guest application. Since it maps from VA.
PMAP is per guest OS

Extended page table - per VM or per guest.

A VM has well defined PA space

Two ways of VMM

Build it into operating system.
OS like explicitly running.

ESX server(special purpose VMM) vs ESX workstation (general purpose os)


Double Paging - two different pieces of software working on it.

Force guest OS to use it's own paging algorithm



#### information on exceptions and interrupt handlers
http://cse.unl.edu/~goddard/Courses/CSCE351/IntelArchitecture/IntelInterupts.pdf
