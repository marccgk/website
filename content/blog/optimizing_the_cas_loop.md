---
# vim: set textwidth=120:

title: "Optimizing the CAS loop"
date: 2018-12-10T10:02:58-08:00
draft: false
---
In the [last post]({{< ref "/blog/a_virtual_memory_linear_arena.md" >}}), I explored a multiple producer, single
consumer linear memory buffer that could be used as an almost-infinite memory resource.

I decided to try and check how the arena would work under stress, so I incremented the number of iterations the test
would run per thread to increase thread contention. Even though lock-free, atomic based solutions tend to do better than
their full lock counterparts, operating on atomic variables is not cheap.

## Profiling

Profiling is one of the most important skills needed by any programmer that cares about software quality.

Before we can start profiling, we need something to measure. I added cycle counts to the test function using Window's
high resolution counter function [QueryPerformanceCounter](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644904).

A while ago, Intel made it's software profiling suite [System Studio](https://software.intel.com/en-us/system-studio/choose-download)
free for everyone. I used Intel VTune Amplifier to profile the code using hardware events to direct the optimizations.

## Optimizing

Once I added the high resolution counters I made the test output an average of cycles elapsed per thread. On my machine,
I was getting ~9,000,000 (9 million) cycles / thread. The actual numbers are not that important, right now they're just
a baseline so we know what impact code changes have.

The next step is to profile using VTune, so we know what is taking the most time. Since I'm profiling a fairly small
amount of code and I'm interested on what's going on in the hardware, I used the Microarchitecture Exploration analysis.

Since we need optimized code for profiling, everything ends up inlined and there's no correspondence between samples
taken and source code lines. Therefore we'll have to take a look at the assembly:

<img src="/img/blog/optimizing_the_cas_loop/vmla_test_00_unoptimized.png" />

The instruction where the most clockticks were assigned is the `jnz` at the end of **Block 5**. That corresponds to the
atomic compare and swap operation that tries to update the `used` variable if and only if it hasn't changed since we
last read it ([vmla.c:95](https://gist.github.com/marccgk/f44a4008bfad6dec18079f3b9b50038c#file-_vmla-c-L95)).
Since all threads are trying to perform this operation at the same time, there's a lot of contention, and
the CPU is constantly waiting for memory (Back-End Bound) since atomic operations need to syncronize CPU cache memory.

This atomic compare and swap operations is only performed in the case where we know there's enough allocated memory,
which is true most of the time. In case it's not true, one thread will allocate more memory, while the rest continue
trying to get allocated memory, causing further contention on the same operation.

There's a simple transform we can apply to the algorithm to reduce contention: we can *always* increase the amount of
`used` memory beforehand ([vmla.c:131](https://gist.github.com/marccgk/f44a4008bfad6dec18079f3b9b50038c#file-_vmla-c-L131)),
since we expect the arena to have *unlimited* memory and, later, check whether we need to
allocate more physical memory to satisfy the allocation request.

Running the test again, it was now reporting a total execution time of ~4.2 million cycles, or less than half the time
of the original implementation.

A second profile using VTune's Microarchitecture Exploration analysis shows a different picture: **Block 3** has
increased in size and it now contains the first atomic operation: an atomic add that increments the amount of `used`
memory in the arena. This is where most of the time is spent now, since *all* threads are contending for memory here
(cache synchronization), but since the addition is inconditional, no thread will try more than once to increment the
variable.

**Block 5**, which contains the mutual exclusion section doesn't even register clocktics. It is expected that most of
the time there's going to be enough memory allocated, so very few operations will enter the CAS loop compared to the
total of number of allocation requests.

<img src="/img/blog/optimizing_the_cas_loop/vmla_test_01_optimized.png" />

## Virtual Memory Linear Arena, Optimized

I uploaded the optimized version to a github gist:

{{< gist marccgk 077e015acfc2c0b1299030699afba615 >}}

This other gist contains both versions, and the test code with high resolution counters:

{{< gist marccgk f44a4008bfad6dec18079f3b9b50038c >}}

