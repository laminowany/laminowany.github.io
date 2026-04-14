---
title: Navigating `pmap` output
description: Making sense of whatever `pmap` spits at you
date: 2026-04-05T23:12:06+02:00
image: cover.png
---

Regardless of whether you need to analyze your process memory with `pmap` or you just enjoy staring at its output in your free time, it's definitely helpful being able to understand its output.
There is of course `man pmap` but it does not provide that much of helpful information ([LINK](https://man7.org/linux/man-pages/man1/pmap.1.html)).
In this post I present brief overview of what `pmap` is trying to communicate.

## Running `pmap`
`pmap` stands for **process map**, it takes PID of a process and displays its memory mappings.
It's common to use it alongside `pgrep` which finds PID of a process:
```sh
pmap $(pgrep ssh)
```
But I often launch my program and `pmap` in one line:
```sh
./program & pmap $!
```
 (`$!` is a bash variable that stores PID of the last background process)

## `pmap` output
### Default output
If you run `pmap` without any additional flags, you will get something like:

```sh
$ ./program & pmap $!

1602974:   ./program
00005a3296039000      4K r---- program
00005a329603a000      4K r-x-- program
00005a329603b000      4K r---- program
00005a329603c000      4K r---- program
00005a329603d000      4K rw--- program
00005a32a2076000    132K rw---   [ anon ]
00007de1fae00000    160K r---- libc.so.6
00007de1fae28000   1620K r-x-- libc.so.6
00007de1fafbd000    352K r---- libc.so.6
00007de1fb015000      4K ----- libc.so.6
00007de1fb016000     16K r---- libc.so.6
00007de1fb01a000      8K rw--- libc.so.6
00007de1fb01c000     52K rw---   [ anon ]
00007de1fb17a000     12K rw---   [ anon ]
00007de1fb185000      8K rw---   [ anon ]
00007de1fb187000      8K r---- ld-linux-x86-64.so.2
00007de1fb189000    168K r-x-- ld-linux-x86-64.so.2
00007de1fb1b3000     44K r---- ld-linux-x86-64.so.2
00007de1fb1bf000      8K r---- ld-linux-x86-64.so.2
00007de1fb1c1000      8K rw--- ld-linux-x86-64.so.2
00007ffe2cbf9000    132K rw---   [ stack ]
00007ffe2cdcb000     16K r----   [ anon ]
00007ffe2cdcf000      8K r-x--   [ anon ]
 total             2776K
```

This shows the very basic information about memory layout. Each row represent a single [**virtual memory area (VMA)**](https://litux.nl/mirror/kerneldevelopment/0672327201/ch14lev1sec2.html) and you can see its:
- starting adress
- size
- permissions
- name

### Extended output
It's not much but it's honest work. To get more information run with `-XX` flag for extended output which will produce something like this:

<a id="sekcja-python"></a>
```sh
$ ./program & pmap $! -XX


1603042:   ./program
     Address Perm   Offset Device  Inode Size KernelPageSize MMUPageSize  Rss Pss Pss_Dirty Shared_Clean Shared_Dirty Private_Clean Private_Dirty Referenced Anonymous KSM LazyFree AnonHugePages ShmemPmdMapped FilePmdMapped Shared_Hugetlb Private_Hugetlb Swap SwapPss Locked THPeligible              VmFlags Mapping
5b494f30e000 r--p 00000000  08:30 214513    4              4           4    4   0         0            4            0             0             0          4         0   0        0             0              0             0              0               0    0       0      0           0       rd mr mw me sd program
5b494f30f000 r-xp 00001000  08:30 214513    4              4           4    4   0         0            4            0             0             0          4         0   0        0             0              0             0              0               0    0       0      0           0    rd ex mr mw me sd program
5b494f310000 r--p 00002000  08:30 214513    4              4           4    4   0         0            4            0             0             0          4         0   0        0             0              0             0              0               0    0       0      0           0       rd mr mw me sd program
5b494f311000 r--p 00002000  08:30 214513    4              4           4    4   4         4            0            0             0             4          4         4   0        0             0              0             0              0               0    0       0      0           0    rd mr mw me ac sd program
5b494f312000 rw-p 00003000  08:30 214513    4              4           4    4   4         4            0            0             0             4          4         4   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd program
5b496f0ca000 rw-p 00000000  00:00      0  132              4           4    4   4         4            0            0             0             4          4         4   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd [heap]
77ed78800000 r--p 00000000  08:30  63709  160              4           4  160   2         0          160            0             0             0        160         0   0        0             0              0             0              0               0    0       0      0           0       rd mr mw me sd libc.so.6
77ed78828000 r-xp 00028000  08:30  63709 1620              4           4  864  11         0          864            0             0             0        864         0   0        0             0              0             0              0               0    0       0      0           0    rd ex mr mw me sd libc.so.6
77ed789bd000 r--p 001bd000  08:30  63709  352              4           4   64   0         0           64            0             0             0         64         0   0        0             0              0             0              0               0    0       0      0           0       rd mr mw me sd libc.so.6
77ed78a15000 ---p 00215000  08:30  63709    4              4           4    0   0         0            0            0             0             0          0         0   0        0             0              0             0              0               0    0       0      0           0          mr mw me sd libc.so.6
77ed78a16000 r--p 00215000  08:30  63709   16              4           4   16  16        16            0            0             0            16         16        16   0        0             0              0             0              0               0    0       0      0           0    rd mr mw me ac sd libc.so.6
77ed78a1a000 rw-p 00219000  08:30  63709    8              4           4    8   8         8            0            0             0             8          8         8   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd libc.so.6
77ed78a1c000 rw-p 00000000  00:00      0   52              4           4   20  20        20            0            0             0            20         20        20   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd
77ed78c13000 rw-p 00000000  00:00      0   12              4           4    8   8         8            0            0             0             8          8         8   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd
77ed78c1e000 rw-p 00000000  00:00      0    8              4           4    4   4         4            0            0             0             4          4         4   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd
77ed78c20000 r--p 00000000  08:30  63706    8              4           4    8   0         0            8            0             0             0          8         0   0        0             0              0             0              0               0    0       0      0           0       rd mr mw me sd ld-linux-x86-64.so.2
77ed78c22000 r-xp 00002000  08:30  63706  168              4           4  168   2         0          168            0             0             0        168         0   0        0             0              0             0              0               0    0       0      0           0    rd ex mr mw me sd ld-linux-x86-64.so.2
77ed78c4c000 r--p 0002c000  08:30  63706   44              4           4   40   0         0           40            0             0             0         40         0   0        0             0              0             0              0               0    0       0      0           0       rd mr mw me sd ld-linux-x86-64.so.2
77ed78c58000 r--p 00037000  08:30  63706    8              4           4    8   8         8            0            0             0             8          8         8   0        0             0              0             0              0               0    0       0      0           0    rd mr mw me ac sd ld-linux-x86-64.so.2
77ed78c5a000 rw-p 00039000  08:30  63706    8              4           4    8   8         8            0            0             0             8          8         8   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me ac sd ld-linux-x86-64.so.2
7ffed533a000 rw-p 00000000  00:00      0  132              4           4   16  16        16            0            0             0            16         16        16   0        0             0              0             0              0               0    0       0      0           0 rd wr mr mw me gd ac [stack]
7ffed53e9000 r--p 00000000  00:00      0   16              4           4    0   0         0            0            0             0             0          0         0   0        0             0              0             0              0               0    0       0      0           0 rd mr pf io de dd sd [vvar]
7ffed53ed000 r-xp 00000000  00:00      0    8              4           4    4   0         0            4            0             0             0          4         0   0        0             0              0             0              0               0    0       0      0           0 rd ex mr mw me de sd [vdso]
                                         ==== ============== =========== ==== === ========= ============ ============ ============= ============= ========== ========= === ======== ============= ============== ============= ============== =============== ==== ======= ====== ===========
                                         2776             92          92 1420 115       100         1320            0             0           100       1420       100   0        0             0              0             0              0               0    0       0      0           0 KB
```

Now we are talking! We got way more information, our columns have meaningful descriptions and some names of **VMAs** got decrypted for us, like [anon] into [vdso].

### Most useful parts of output

However, most of this information is quite useless, so usually I focus only those columns:
```sh
1603042:   ./program
     Address Perm   Offset Device  Inode Size  Rss Anonymous  Mapping
5b494f30e000 r--p 00000000  08:30 214513    4    4         0  program
5b494f30f000 r-xp 00001000  08:30 214513    4    4         0  program
5b494f310000 r--p 00002000  08:30 214513    4    4         0  program
5b494f311000 r--p 00002000  08:30 214513    4    4         4  program
5b494f312000 rw-p 00003000  08:30 214513    4    4         4  program
5b496f0ca000 rw-p 00000000  00:00      0  132    4         4  [heap]
77ed78800000 r--p 00000000  08:30  63709  160  160         0  libc.so.6
77ed78828000 r-xp 00028000  08:30  63709 1620  864         0  libc.so.6
77ed789bd000 r--p 001bd000  08:30  63709  352   64         0  libc.so.6
77ed78a15000 ---p 00215000  08:30  63709    4    0         0  libc.so.6
77ed78a16000 r--p 00215000  08:30  63709   16   16        16  libc.so.6
77ed78a1a000 rw-p 00219000  08:30  63709    8    8         8  libc.so.6
77ed78a1c000 rw-p 00000000  00:00      0   52   20        20 
77ed78c13000 rw-p 00000000  00:00      0   12    8         8 
77ed78c1e000 rw-p 00000000  00:00      0    8    4         4 
77ed78c20000 r--p 00000000  08:30  63706    8    8         0  ld-linux-x86-64.so.2
77ed78c22000 r-xp 00002000  08:30  63706  168  168         0  ld-linux-x86-64.so.2
77ed78c4c000 r--p 0002c000  08:30  63706   44   40         0  ld-linux-x86-64.so.2
77ed78c58000 r--p 00037000  08:30  63706    8    8         8  ld-linux-x86-64.so.2
77ed78c5a000 rw-p 00039000  08:30  63706    8    8         8  ld-linux-x86-64.so.2
7ffed533a000 rw-p 00000000  00:00      0  132   16        16  [stack]
7ffed53e9000 r--p 00000000  00:00      0   16    0         0  [vvar]
7ffed53ed000 r-xp 00000000  00:00      0    8    4         0  [vdso]
                                         ==== ==== ========= 
                                         2776 1420       100  KB
```

### Fields explained

Each memory areas has:
- **Address** - this denoted where the **VMA** starts, please notice that it's aligned to page boundaries (usually 4096 bytes)
- **Perm** - permissions for given **VMA**:
  - Read
  - Write
  - eXecutable
  - Private / Shared
    - If mapping is **Shared** then changes to given memory page are visible to other processes as well. If mapping is **Private** then on memory modification a copy of page will be created ([Copy-On-Write](https://en.wikipedia.org/wiki/Copy-on-write) pattern) 
- **Offset** - only relevant if **VMA** is file-backed. Then it denotes the offset since beginning of the file that this VMA is mapped into.
- **Device** - device in format *(major:minor)* where **major** refers to specific driver and **minor** to disk/partiion. If it's zero than VMA is not associated with any file.
  - For more info look here: [devices.txt](https://github.com/torvalds/linux/blob/master/Documentation/admin-guide/devices.txt).
- **Inode** represent an ID of file in Linux. This indicates if VMA is mapped to a file and if so to which one. If you need to localize given file by its **Inode** you can run `find / -inum <INODE>` (preferably with sudo to avoid tons of warnings about permissions).
- **Size** and **Rss** (Resident Set Size) - **Size** is how much memory the kernel **reserved** for that VMA and **RSS** is the total size of memory frames that actually got allocated for given VMA.
  - Caveat: Two processes can map the same area of physical memory into their virtual address spaces, but the kernel can load given area only once to physical memory. So if process A has some segment with RSS=8 and process B maps the same segment with RSS=8 it does not mean that 16KB of physical memory is taken, as kernel will have both those mapping point to one physical location. If you want to check the actual pressure that given VMA is putting on physical memory look for **Pss** column [(in previous listing)](#extended-output), which as the name suggests (**Proportional Set Size**) calculates the share of given process by dividing RSS by number of processes which have mapped the same area of physical memory.
- **Anonymous** refers to the size of memory that is not mapped to any file (hence anonymous). OS has no intention of writing modifications of this memory to any file, this is usually operational memory for a program, like stack or heap.
- **Mapping** is a human friendly name of given section. It's often name of a file or library that **VMA** is mapped to, or specific name of programs data structure like **[heap]** or **[stack]**.

### Program sections

You can notice that both the program as well as the needed libraries (libc.so.6, ld-linux-x86-64.so.2) occupy multiple VMAs:

```sh
     Address Perm   Offset Device  Inode Size  Rss Anonymous  Mapping
5b494f30e000 r--p 00000000  08:30 214513    4    4         0  program
5b494f30f000 r-xp 00001000  08:30 214513    4    4         0  program
5b494f310000 r--p 00002000  08:30 214513    4    4         0  program
5b494f311000 r--p 00002000  08:30 214513    4    4         4  program
5b494f312000 rw-p 00003000  08:30 214513    4    4         4  program
(...)
```

Why is that? The reason is simple: different sections of binary are mapped into different VMAs.
You can see it by looking into the `Inode` and `Offset` columns.
`Inode` is the same for all VMAs, while `Offset` is different. It means that that different parts of the same file are mapped into various VMAs.

But how to decipher which sections are loaded where? And how does kernel know which sections of binary to load?

All this information is stored inside ELF binary in program headers. You can display them by running:
`readelf <BINARY> -l`

```sh
$ readelf program  -l

Elf file type is DYN (Position-Independent Executable file)
Entry point 0x1080
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000660 0x0000000000000660  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x00000000000001a1 0x00000000000001a1  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000fc 0x00000000000000fc  R      0x1000
  LOAD           0x0000000000002db0 0x0000000000003db0 0x0000000000003db0
                 0x0000000000000260 0x0000000000000268  RW     0x1000
  DYNAMIC        0x0000000000002dc0 0x0000000000003dc0 0x0000000000003dc0
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  NOTE           0x0000000000000368 0x0000000000000368 0x0000000000000368
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  GNU_EH_FRAME   0x000000000000201c 0x000000000000201c 0x000000000000201c
                 0x0000000000000034 0x0000000000000034  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000002db0 0x0000000000003db0 0x0000000000003db0
                 0x0000000000000250 0x0000000000000250  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash 
          .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt
   03     .init .plt .plt.got .plt.sec .text .fini
   04     .rodata .eh_frame_hdr .eh_frame
   05     .init_array .fini_array .dynamic .got .data .bss
   06     .dynamic
   07     .note.gnu.property
   08     .note.gnu.build-id .note.ABI-tag
   09     .note.gnu.property
   10     .eh_frame_hdr
   11
   12     .init_array .fini_array .dynamic .got
```

That's a lot of information, but we care only about headers `LOAD` and `GNU_RELRO` which leaves us with:

```sh
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000660 0x0000000000000660  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x00000000000001a1 0x00000000000001a1  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000fc 0x00000000000000fc  R      0x1000
  LOAD           0x0000000000002db0 0x0000000000003db0 0x0000000000003db0
                 0x0000000000000260 0x0000000000000268  RW     0x1000
  GNU_RELRO      0x0000000000002db0 0x0000000000003db0 0x0000000000003db0
                 0x0000000000000250 0x0000000000000250  R      0x1

 Section to Segment mapping:
  Segment Sections...
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash 
          .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt
   03     .init .plt .plt.got .plt.sec .text .fini
   04     .rodata .eh_frame_hdr .eh_frame
   05     .init_array .fini_array .dynamic .got .data .bss
   12     .init_array .fini_array .dynamic .got
```

`LOAD` headers instruct which sections of binary are loaded as part of which segment.
You can see that there are four `LOAD` headers which will result in four VMAs.
There is also `GNU_RELRO` header which stands for Relocation Read-Only.
Explanation of this header is beyond scope of this blog post but you can read more about it [here in Redhat docs](https://www.redhat.com/en/blog/hardening-elf-binaries-using-relocation-read-only-relro). 
All you need to know about `GNU_RELRO` is that it changes the permission of some VMA from **Read&Write** to **Read Only**, effectively splitting it into two distinct VMAs.
And this is the reason that while our program has five VMAs despite only four `LOAD` headers being present.

With all that knowledge we can easily match the sections of program from `readelf` output to VMAs presented by `pmap`, matching them by `Offset` and permission `Flags`:
```diff
       Address Perm   Offset Device  Inode Size  Rss Anonymous  Mapping
  5b494f30e000 r--p 00000000  08:30 214513    4    4         0  program
+ .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash
+       .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt   
  5b494f30f000 r-xp 00001000  08:30 214513    4    4         0  program
+                              .init .plt .plt.got .plt.sec .text .fini
  5b494f310000 r--p 00002000  08:30 214513    4    4         0  program
+                                       .rodata .eh_frame_hdr .eh_frame
  5b494f311000 r--p 00002000  08:30 214513    4    4         4  program
+                                 .init_array .fini_array .dynamic .got
  5b494f312000 rw-p 00003000  08:30 214513    4    4         4  program
+                                                            .data .bss
```

Note that while headers in ELF can specify arbitraty offset (i.e. `0x2db0`), the section is always loaded in process memory into page boundaries, so it will be loaded starting from `0x02000` as you can see in `pmap` output.
You can check your page size by running `getconf PAGESIZE`. The default is usually 4KB which is `0x1000` in hex. This explains why all offsets of VMAs are multiples of `0x1000`.

Explaining each segment is out of scope for this blog post, but maybe I will go into more details in the future if there is interest.

#### Terminology alert 
There is important distinction between **sections** and **segments**, and these term should not be used interchangeably. 
**Sections** are parts of your binary, like `.text` or `.rodata`. They are created during compilation and linking, and are most often regarded in that context.
**Segments** are what kernel is concerned with, when starting process and loading binary into memory.


To sum up: when you compile your program, the compiler creates a various **sections** of your program. Then, linker packs these **sections** into **segments** and saves them into binary. When the program starts, kernel loads these **segments** into **VMA (Virtual Memory Area)**.

##  How does `pmap` know all this

The answer to that question is *everything is a file*.

Each process on Linux has its own directory `/proc/<PID>`. I highly recommend checking it out if you need to kill your time during slow weekend, it's a gold mine for cool things and understanding how the OS works.

`pmap` looks specifically into:
- `/proc/<PID>/maps`
- `/proc/<PID>/smaps`


## ASLR

Did you wonder why everytime you run program you get different addresses for each VMA? 

The answer lies behind **ASLR** - [Address Space Layout Randomization](https://en.wikipedia.org/wiki/Address_space_layout_randomization).
It's a security feature which randomly arranges VMAs arround memory, so it's harder for the attacker to know which section lands where.
If you wan't to disable it for scientific purposes (I'm not gonna blame you), then you can run `echo 0 | sudo tee /proc/sys/kernel/randomize_va_space`.

To read more on ASLR: [Ubuntu docs](https://documentation.ubuntu.com/security/security-features/process-memory/aslr/).

## Summary

I hope you enjoyed this article. `pmap` is a great tool to understand the memory layout of a process on Linux.

If you are interested in more details on this topic you might like my talk from NDC Techtown 2024: [Demystifying Process Address Space: Heap, Stack, and Beyond](https://youtu.be/pI-HRRh7-dU).

-----

