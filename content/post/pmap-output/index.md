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

It's not much but it's honest work. To get more information run with `-XX` flag for extended output which will produce something like this:

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

However, most of this information is quite useless, so usually I focus only those columns:
```
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

Each memory areas has:
- **Address** - this denoted where the **VMA** starts, please notice that it's aligned to page boundaries (usually 4096 bytes)
- **Perm** - permissions for given **VMA**:
  - Read
  - Write
  - eXecutable
  - Private / Shared
- **Offset** - this is only relevant if **VMA** maps to some file. Then it means the offset since beginning of the file that this VMA is mapped into.

Don't confuse **segments** vs **section**  vs **areas** terminology.

where does pmap takes it from?

how to know which sectioons are loaded from where

-----

