---
title: BI-SOJ Real mode debug setup
tags: gdb qemu soj assembly school

---

## Introduction

For BI-SOJ course in school, I needed to setup debugging for running Boot sectors on Qemu in Real Mode. Soon, I realised it wasn’t going to be as straightforward and simple as I originally thought. Thus, after I spent some time reasearching, I decided to put together a simple tutorial to save some time in future.

Please note that this tutorial merely puts together work of other authors. Splendid `.gdbinit` by [https://ternet.fr/gdb_real_mode.html](https://ternet.fr/gdb_real_mode.html) and `target.xml` found [here](https://gist.githubusercontent.com/MatanShahar/1441433e19637cf1bb46b1aa38a90815/raw/2687fb5daf60cf6aa8435efc8450d89f1ccf2205/target.xml).
<!--more-->

## TL;DR

download [`target.xml`](https://gist.githubusercontent.com/MatanShahar/1441433e19637cf1bb46b1aa38a90815/raw/2687fb5daf60cf6aa8435efc8450d89f1ccf2205/target.xml),[`.gdbinit`](https://github.com/mhugo/gdb_init_real_mode) and save them to the folder used to launch `gdb`.

## Step By Step

In order to be able to successfully debug my bootsector program, I run `qemu` in one terminal and `gdb` in another.

First, I run qemu with `qemu-system-i386 -s -S`

```
-s  starts gdbserver session on port 1234
-S  waits for the gdb client to attach, does not launch code when run
```
This initiates the qemu simulator and opens `gdb` stub for incoming debugger sessions on port 1234. It also stops and waits for gdb to be attached.

The next step is to run gdb in a separate terminal window:

```bash
$ gdb -nh -ex 'source .gdbinit' -ex 'set tdesc filename target.xml' -ex 'b *0x7c00'
```

```bash
	-nh prevents gdb from using ~/.gdbinit file 
	-ex 'source .gdbinit' loads our downloaded ./.gdbinit 
	-ex 'set tdesc filename target.xml' loads target description from ./target.xml 
	-ex 'b *0x7c00' sets breakpoint to start of our bootsector
```
This command starts `gdb`, performs commands specified by `-ex` switches and waits for next commands.

The next step is to connect to the `gdb` stub created by qemu. This is done by the following:

```
(gdb) target remote :1234
```

We should now see that code execution stopped at the beginning of bios code. All we have to do now, to get to our code is:

```
(gdb) c
```
As we previously set the breakpoint to `0x7c00`, the command `c` continues the execution until this breakpoint is hit.

## More Comfortable way

one can also create a gdb script to pass to gdb to run all the commands upon start.

```bash
/* my_gdbinit.gdb */
source .gdbinit
set tdesc filename target.xml
b *0x7c00
target remote :1234
```

Save this to file `my_gdbinit.gdb` , start `qemu` listening to gdb connections on :1234 and then run 

```bash
$ gdb -nh -ex 'source my_gdbinit.gdb'  
```

## Why like this

Since gdb doesn’t support real mode by default (segmentation is ignored, ..), extra work is needed to alter it’s behavior to support it (to see the correct disassembly), as explained in [this blogpost](https://ternet.fr/gdb_real_mode.html). This is however not sufficient for newer versions of gdb, as explained in [this post](https://www.bountysource.com/issues/55548690-set-architecture-i8068-has-no-effect-on-disassembly). The circumvention then is to load our own target description file.

## Conclusion

Putting this whole debugging procedure together took me like a day and I thought someone might find it useful. Have fun!

## References:
[https://ternet.fr/gdb_real_mode.html](https://ternet.fr/gdb_real_mode.html)

[https://sourceware.org/gdb/current/onlinedocs/gdb/Target-Descriptions.html](https://sourceware.org/gdb/current/onlinedocs/gdb/Target-Descriptions.html) - target description reference