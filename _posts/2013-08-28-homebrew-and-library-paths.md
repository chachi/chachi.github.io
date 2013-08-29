---
layout: post
title: Homebrew and Library Search Paths
---

[Homebrew][homebrew] really is everything [Fink][fink] and [MacPorts][macports] wish they could be. It works well, it's easy to modify, and, for the most part, it fits right into your workflow without any changes. Using programs installed by Homebrew doesn't require you to change your `$PATH` since /usr/local/bin is already included and most libraries are also available immediately in /usr/local/lib.

Unfortunately, there are a few gotchas hidden in this fantastic tool. My current project with [RPG][rpg] uses GCC 4.7 to build C++ on OSX. Installing it was, of course, a cinch with Homebrew and I could use it immediately without any other system changes. I had no problems for about two weeks until suddenly [Google Protocol Buffers][protobuf] started to fail calling `std::string::assign()`.

Here's the error message:

    SlamMapTest(98263) malloc:
    *** error for object 0x104ce8820: pointer being freed was not allocated
    *** set a breakpoint in malloc_error_break to debug

This was baffling and to summarize a few hours of searches, I finally tracked it down to being related to [this recent change][gotcha] to Apple's libstdc++. What was sadly occuring here is that while GCC 4.7 was linked into /usr/local/bin *none of its corresponding system libraries* were linked or copied into /usr/local/lib. This meant that I was **compiling** my software using GCC 4.7, but **linking** against Apple's libstdc++.dylib. This resulted in the confusing mismatch in dynamic string usage.

<s>To solve this, I simply added</s>

<s>```bash
export DYLD_LIBRARY_PATH = /usr/local/Cellar/gcc47/4.7.3/gcc/lib
```
</s>

<s>to any script that runs my executables. This causes the correct libraries to be found when I run something built with GCC 4.7 and solves the issue. Unforunately, I can't set the system path to this because other executables depend on using the system library.</s>

<s>**EDIT:** Corrected the last paragraph on when `DYLD_LIBRARY_PATH` is actually set.</s>

**EDIT:** It turns out the actual fix is to rebuild GCC and pass `--enable-fully-dynamic-string` to `./configure`. This links your executables to the correct libstdc++.dylib and makes the interoperable with the system libraries. Use `brew edit gcc47` to edit the Homebrew formula to add this configure option.

[homebrew]: http://brew.sh
[fink]: http://finkproject.org
[macports]: http://macports.org
[rpg]: http://rpg.robotics.gwu.edu
[protobuf]: https://developers.google.com/protocol-buffers/
[gotcha]: http://newartisans.com/2009/10/a-c-gotcha-on-snow-leopard/
