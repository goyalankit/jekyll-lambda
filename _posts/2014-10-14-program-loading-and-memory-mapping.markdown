---
layout: post
title: "program loading and memory mapping"
date: 2014-10-14 15:40:27 -0500
comments: true
categories: os lab memory loading
---

`arch_prctl(ARCH_SET_FS, addr)`
---

From man page:
> set architecture specific thread state

- `ARCH_SET_FS`: set the 64-bit base address for the `FS` register to `addr`

    Earlier kernel used to store an array indexed by threadid containing address of thread space, however now it sets the FS register for threads local storate.
    `GS` is used for kernel threads storage pointer whereas `FS` is used for userspace threads.

<!-- more -->

ELF Format:
---

Creating the memory image of a new process
---

sys_execve is responsible for setting up the environment for running the program. Below are the steps taken by sys_execve which calls do_execve_common:

1. Check that `NPROC` limit is not exceeded (i.e.,total number of process), if it is then exit.(`L:1443`)
2. Allocate memory for data structure in kernel.(`L:1458`)
3. Open the exec file using `do_open_exec`(`L:1469`)
4. Now the kernel data structures are initialized and `exec_binprm` is called.
5. `exec_binprm` calls `search_binary_handler` which finds the binary format handler, in our case elf. So it finds load_elf_binary. (fs/binfmt_elf.c L:84 and 571)
    * `load_elf_binary` does consistency checks by making sure that it’s an ELF format file by comparing the main number and ELF in `e_ident` field in header.
    * `load_elf_binary` reads the header information and looks for `PT_INTERP` segment to see if an inter- preter was specified. This segment is only present for dynamically linked programs and not for statically linked.
    * Now all the loadable segments are mmapped into memory, by reading the ELF Program headers. `bss` segment is also mapped.
    * `create_elf_tables` creates a stack at a random offset and sets the auxiliary vectors, arguments and environments according to the standard.
    * Finallythecontrolistransferredtoe_entrypointusingstart_threadmethod.(`fs/binfmt_elf.cL:990`)

<img src="{{root_url}}/images/post-images/elf.png"/>
source: Professional Linux Kernel Architecture

<img src="{{root_url}}/images/post-images/elf2.png"/>
source: stackoverflow.com

Change the .text entry address in ELF
---

```
gcc test1.c -o test1.out -Wl,-Ttext-segment=0x2000000 -static
```

http://stackoverflow.com/questions/8116648/why-is-the-elf-entry-point-0x8048000-not-changeable?lq=1

# entry point vs Load Address

First LOAD specifies the program code in the file.
start(glibc) is the entry point and not the address in LOAD.

```
start -> init -> main
```


Stack Growth
---

<img src="http://i.imgur.com/ynPxqhZ.png?2"/>



Links for ELF
---
1. http://www.skyfree.org/linux/references/ELF_Format.pdf
2. http://articles.manugarg.com/aboutelfauxiliaryvectors.html
