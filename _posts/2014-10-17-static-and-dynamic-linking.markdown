---
layout: post
title: "static and dynamic linking"
date: 2014-10-17 19:25:26 -0500
comments: true
categories: linking compilers
---


```C
// add.h
int add(int, int);

// sub.h
int sub(int, int);

```
Create the corressponding c files.

```C
// add.c
int add(int i, int j) {
  return i + j;
}

// sub.c
int sub(int i, int j) {
  return i - j;
}
```

<!-- more -->

```
// demo.c
#include <stdio>
#include "add.h"
#include "sub.h"

printf("Sum of %d and %d is %d", i, j, add(i, j));
printf("Difference of %d and %d is %d", i, j, sub(i, j));
```

Static Libraries
---

```sh
$ gcc -c add.c
$ gcc -c sub.o

$ ar rs libheymath.a add.o sub.o

$ gcc -c demo.c
$ gcc -o demo.out -L . demo.o -lheymath

```

### .symtab

```sh
    Num:    Value          Size Type    Bind   Vis      Ndx Name
    55: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    56: 0000000000601038     0 OBJECT  GLOBAL HIDDEN    24 __dso_handle
    57: 000000000040058c    20 FUNC    GLOBAL DEFAULT   13 sum
    58: 0000000000400640     4 OBJECT  GLOBAL DEFAULT   15 _IO_stdin_used
    59: 00000000004005c0   101 FUNC    GLOBAL DEFAULT   13 __libc_csu_init
    60: 0000000000601048     0 NOTYPE  GLOBAL DEFAULT   25 _end
    61: 0000000000400440     0 FUNC    GLOBAL DEFAULT   13 _start
    62: 0000000000601040     0 NOTYPE  GLOBAL DEFAULT   25 __bss_start
    63: 000000000040052d    95 FUNC    GLOBAL DEFAULT   13 main
    64: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses
    65: 0000000000601040     0 OBJECT  GLOBAL HIDDEN    24 __TMC_END__
    66: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    67: 00000000004005a0    22 FUNC    GLOBAL DEFAULT   13 sub
    68: 00000000004003e0     0 FUNC    GLOBAL DEFAULT   11 _init
```

Dynamic Libraries
---

Dynamic libraries need to be created explicitly. They take less space since they can be shared and used functions are not put in executable.

Note that dynamic libraries introduce dependencies and the executable cannot be run directly on another system.

```sh
$ gcc -Wall -fPIC -c add.c
$ gcc -Wall -fPIC -c sub.c

$ gcc -shared -o libheymath.so add.o sub.o
$ sudo cp libheymath.so /usr/lib/
$ ldconfig /usr/lib/libheymath.so
$ gcc -o demo.out demo.o -lheymath
```

Dependencies can be checked using `ldd`

```
$ ldd demo.out

	linux-vdso.so.1 =>  (0x00007fffe5bfe000)
	libheymath.so => /usr/lib/libheymath.so (0x00007fdc19d77000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fdc199b1000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fdc19f95000)
```

## .symtab

Notice that the sub and add are have undefined index and will be resolved at runtime.

```sh
    Num:    Value          Size Type    Bind   Vis      Ndx Name
    53: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    54: 0000000000601048     0 OBJECT  GLOBAL HIDDEN    24 __dso_handle
    55: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sum
    56: 0000000000400800     4 OBJECT  GLOBAL DEFAULT   15 _IO_stdin_used
    57: 0000000000400780   101 FUNC    GLOBAL DEFAULT   13 __libc_csu_init
    58: 0000000000601058     0 NOTYPE  GLOBAL DEFAULT   25 _end
    59: 0000000000400630     0 FUNC    GLOBAL DEFAULT   13 _start
    60: 0000000000601050     0 NOTYPE  GLOBAL DEFAULT   25 __bss_start
    61: 000000000040071d    95 FUNC    GLOBAL DEFAULT   13 main
    62: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses
    63: 0000000000601050     0 OBJECT  GLOBAL HIDDEN    24 __TMC_END__
    64: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    65: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sub
    66: 00000000004005b0     0 FUNC    GLOBAL DEFAULT   11 _init
```

References:
---

1. http://cs-fundamentals.com/c-programming/static-and-dynamic-linking-in-c.php
