---
# vim: set textwidth=120:

title: "A Virtual Memory Linear Arena"
date: 2018-11-19T16:15:25-08:00
draft: false
---

A linear memory arena, or allocator, is probably the most useful memory allocation strategy. A simple implementation combined
with the temporal and spatial locality inherent in most memory usage makes it a must in any programmer's toolbox.

Generally, linear arenas take the shape of a dynamically growing array of elements, e.g. Sean Barrett's [stretchy
buffer](https://github.com/nothings/stb/blob/master/stretchy_buffer.h) in C or
[std::vector](https://en.cppreference.com/w/cpp/container/vector) in C++. This is all good if your linear allocation
needs happen to be restricted to a single type. When this is not the case, we need a different strategy.

In my case, I wanted the arena to have the following properties:

1. _Unlimited_ memory: grow dynamically with use, no preallocated memory.
2. Stable addresses: don't copy old memory into a new address range when growing.
3. Multiple producer: support multiple threads pushing data at the same time.

Properties #1 and #2 can be achieved by leveraging the [virtual
memory](https://en.wikipedia.org/wiki/Virtual_memory) system of any modern OS.

Property #3 needs some kind of shared memory synchronization strategy like
[mutual exclusion](https://en.wikipedia.org/wiki/Lock_(computer_science)) or
[critical sections](https://en.wikipedia.org/wiki/Critical_section).

## Virtual Memory

Using virtual memory it's possible to _reserve_ a huge amount of memory (more
than what's available in a given machine) without actually _allocating_ it.

The virtual memory system will assign a _continuous block_ of memory to the
reservation, which satisfies property #2.

In order to use the reserved memory, we need to _commit_ it, an operation that will
instruct the operating system to back the virtual memory with physical memory
([RAM](https://en.wikipedia.org/wiki/Random-access_memory)) so the application
can read and write to it.

These are the functions from the Windows API (Win32) that we'll use:

* [VirtualAlloc](https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-virtualalloc): reserve and commit virtual memory.
* [VirtualFree](https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-virtualfree): release virtual memory.

## Shared Memory Synchronization

I wanted to avoid a full OS level critical section / mutex lock, so I chose a two level synchronization strategy for this arena.

* If there's enough room to make an allocation, an atomic [compare and swap](https://en.wikipedia.org/wiki/Compare-and-swap)
    pushes the amount of used bytes by the allocation size.
* If the allocation size is too big to fit in the remaining committed memory, the algorithm enters a short critical
    section by locking an atomic variable. There, it commits more memory, exits the critical section and tries again to
    make the allocation.

Note that the critical section only prevents more than one thread to commit virtual memory, it doesn't prevent other
threads to allocate if there are enough bytes left (e.g. an allocation of 64 KB could need to commit more memory, but
an allocation of 24 bytes could succeed while the first thread is busy in the critical section committing more virtual
memory).

## Source Code

I made a gist with a standalone implementation (only dependencies are `windows.h` and `stdint.h`) and a test.
To build it, download the three files and call `build.bat`.  This will generate a `vmla_test.exe` in a `build` folder.
The build script supports Visual Studio 2015 and 2017 (I only tested Professional Edition for 2017).

{{< gist marccgk 524d4d903e10699ca7fe266d96eb0fc7 >}}
