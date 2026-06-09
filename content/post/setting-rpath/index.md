---
title: "Why Setting RPATH Doesn't Actually Set RPATH"
description: How your linker thinks it knows better than you
image: cover.png
date: 2026-06-05T19:03:34+02:00
tags:
    - linux
    - linking
---

If you ever dabbled with dynamic linking in Linux and needed to specify where the dynamic loader (ld-linux.so) should look for shared libraries, you probably used
either the  `LD_LIBRARY_PATH` environment variable or set `RPATH` directly in your binaries. So did I. But to my surprise it no longer works as it used to.

## How I thought it always worked

Quick recap: when the dynamic loader loads your executable and tries to resolve its dynamic dependencies, it needs to know where to look for them.

By default, it searches system-wide paths like `/lib` and `/usr/lib`. This can be overridden by setting the `LD_LIBRARY_PATH` environment variable, which takes precedence.
But sometimes you need to embed the path to shared libraries inside the binary, and not depend on environment settings.

Then `RPATH` comes to the rescue. It's an entry in executable binary, which takes precedence over both default systems paths as well as the `LD_LIBRARY_PATH` environment variable.
As a linker flag, it is set using `-Wl,-rpath,<PATH>`.
It's part of binary, it's always there, it does not depend on external settings.

So the order of lookup is:
1. Path written into the binary as an `RPATH` entry
2. Path set by environment variable `LD_LIBRARY_PATH`
3. Default paths (`/lib`, `/usr/lib`)

But recently I tried to use `-rpath` and to my surprise it did not work as expected.

> To be pedantic: order of looking for shared libraries is a little bit more complex than that, you can read details here: [ld-linux.so - Linux manual page](https://man7.org/linux/man-pages/man8/ld.so.8.html)

## Not today...

I had a simple program depending on the Qt framework which consists of many shared libraries. I had multiple versions of Qt installed on my machine, and to link against a specific one, I first used `LD_LIBRARY_PATH`. Everything worked. As expected.

Until I tried to override `LD_LIBRARY_PATH` with `RPATH`. I set the flag in CMake as such:

```cmake
target_link_options(app PRIVATE
    "-Wl,-rpath,/home/Qt/dev"
)
```

But to my surprise, the old version of Qt (the one from  `LD_LIBRARY_PATH`), was still being used. I rebuilt twice. Still the same.
I ditched the CMake and tried calling the compiler directly. It still did not give me the expected effect.

Okay, the next step is to look into the binary directly. Setting `rpath` flag should embed the path into the binary and I should be able to check it by tools like `readelf`.
Passing `-d` to `readelf` should show me information related to **D**ynamic linking.
```sh
$ readelf app -d | grep -E "RPATH" 

<NO OUTPUT>
```

Nothing is there? But how can that be if I just set it?

The next step is to bet on `grep` not working correctly. I list entire dynamic section:
```diff
$ readelf app -d 

Dynamic section at offset 0x2da0 contains 30 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             (...)
+0x000000000000001d (RUNPATH)            Library runpath: [/home/Qt/dev]
 0x000000000000000c (INIT)               0x1000
 0x000000000000000d (FINI)               0x11dc
 0x0000000000000019 (INIT_ARRAY)         0x3d88
 0x000000000000001b (INIT_ARRAYSZ)       16 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x3d98
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x3a0
 0x0000000000000005 (STRTAB)             0x500
 0x0000000000000006 (SYMTAB)             0x3c8
 0x000000000000000a (STRSZ)              403 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x4000
 0x0000000000000002 (PLTRELSZ)           96 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x820
 0x0000000000000007 (RELA)               0x700
 0x0000000000000008 (RELASZ)             288 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffffb (FLAGS_1)            Flags: PIE
 0x000000006ffffffe (VERNEED)            0x6b0
 0x000000006fffffff (VERNEEDNUM)         2
 0x000000006ffffff0 (VERSYM)             0x694
 0x000000006ffffff9 (RELACOUNT)          4
 0x0000000000000000 (NULL)               0x0
```

And while I can see that there is no `RPATH` entry, there is a `RUNPATH` entry with the value I passed via the `-rpath` flag.
What exactly is happening?

## How it actually works

What is the difference between them two?
So while the `RPATH` is a path where to look for shared libraries, `RUNPATH` is similar but less aggressive.

Two differences are:
1. `RUNPATH` has lower priority than `LD_LIBRARY_PATH` environment variable. This is contrary to `RPATH` priority.
2. `RPATH` is inherited across the dependency resolution - if your application **A** uses `RPATH` to look for dependency **B**, then **B** can use this `RPATH` as well to look for its own dependencies. `RUNPATH` does not exhibit this behavior.

So the ACTUAL order of lookup is:
1. Path written into binary as `RPATH` entry (if `RUNPATH` does not exist)
2. Path set by environment variable `LD_LIBRARY_PATH`
3. Path written into binary as `RUNPATH` entry
4. Default paths (`/lib`, `/usr/lib`)

What was not working for me: first I set the `LD_LIBRARY_PATH` environment variable, and then I tried overriding it with `RPATH`.
But instead of setting `RPATH` I ended up with a `RUNPATH` entry which has lower priority than `LD_LIBRARY_PATH`.
As a result the shared libraries were still being loaded from `LD_LIBRARY_PATH` environment variable.

Case closed.

But why did the linker actually set the `RUNPATH` when I explicitly told him to emit `RPATH` using the `-rpath` flag?

Well, this behavior has existed for a long time.
> (since 2013 actually - https://sourceware.org/pipermail/binutils/2013-January/079988.html)

If you want to emit actual `RPATH` you also need to set `--disable-new-dtags` flag.
This instructs the linker to use the "old" behavior of emitting `RPATH` when being asked to emit the damn `RPATH`.

Why did I hit this case now? This must have been the only time when I had the `LD_LIBRARY_PATH` environment variable set in the terminal,
from which I launched my dynamically linked application. Possibly the linker was emitting `RUNPATH` every single time and just did not realize it,
because there was not a `LD_LIBRARY_PATH` set to override it.

Well, the lesson learnt.

## Extra

Both `RPATH` and `RUNPATH` can take a special token - `$ORIGIN` , which is resolved at runtime to the directory containing the executable.
This is useful when deploying applications with shared dependencies as you can set `RUNPATH` to `$ORIGIN/lib` and copy the dependencies along.
It also makes the program relocatable, as long as it is moved alongside its shared dependencies.
This approach is commonly used in bundled application distributions, such as the Qt framework.