---
# vim: set textwidth=120:

title: "Tips on writing code for Data-Oriented Design"
date: 2019-03-27T16:00:00-08:00
draft: false
---

I [recently]({{< ref "/blog/on_cpp_and_object_oriented_programming.md" >}}) wrote about why I think Object Oriented
Programming is not a good tool to write code.

In that post I wrote:

> However, in my experience, OOP is taken as the *gold standard* for software develoment by the majority of professionals.

Object Oriented Programming is often taught and, therefore, learned, right after the basic programming constructs: variables,
conditionals, loops, basic types and, in languages like C++, pointers and basic memory allocation / deallocation. Hence,
most people have only approached medium to large codebases from an OOP lens. Although there are large codebases written
in non OOP languages, e.g. [the Linux Kernel](https://github.com/torvalds/linux), written in C, I'd argue most sizeable
codebases currently in development are built using Object Oriented Design[^1].

In this blog post I'd like to present a different starting point to approach writing software. Obligatory caveat: this
is not a one size fits all solution, nor it doesn't pretend to be. Writing code is still a craft more than a science, so
in my opinion, every "do this, don't do that" advice should be paired with measurable pros and cons.

Having said that, I should also note that as a community, software developers don't even agree on what characteristics
make the list of advantages or disadvantages nor what priorities should be assigned to them.

## Data-Oriented Design

I assume you're familiar with [Mike Acton's](https://twitter.com/mike_acton) [Data-Oriented Design CppCon 2014 talk](https://www.youtube.com/watch?v=rX0ItVEVjHc). If not, go watch it now, I'll wait.

Data-Oriented Design (DOD) is a fundamental concept, understanding how the hardwared works (at a high level) is a
prerequisite to writing instructions for a computer to execute. However, DOD doesn't tell you how to write code. In
[this conversation](http://www.macton.ninja/home/onwhydodisntamodellingapproachatall) with
[Christer Ericson](https://twitter.com/christerericson) that Mike Acton made public, Christer explains why DOD is not a
modelling approach:

> Q: Isn't Data-Oriented Design just dataflow programming?

> Or: Why DoD isn't a modelling approach at all.

> I'll quote an exchange with Christer Ericson's answer (with permission) on this subject:

>> No, not dataflow programming.
>>
>> Dataflow programming, as well as OOD for that matter, is a modelling approach, and specifically for dataflow
>> programming, by expressing data connectivity as a graph.
>>
>> While neither me, nor Mike [Acton], nor Noel [Llopis] has ever provided an “official definition” of DOD (nor have we
>> really been interested in doing so, nor would we necessarily 100% agree on one), I would argue that DOD is not a
>> modelling approach, in fact it’s the opposite thereof.
>>
>> As Mike has eloquently pointed out elsewhere, computation is a transformation of data from one form into another. DOD is
>> a methodology (or just a way of thinking) where we focus on streamlining that transformation by focusing on the input
>> and output data, and making changes to the formats to make the transformation “as light” as possible. (Here there are
>> two definitions of “light.” Mike would probably say that “light” means efficient in terms of compute cycles. I would
>> probably say “light” means in terms of code complexity. They’re obviously related/connected. The truth might be in
>> between.)
>>
>> I say this is the opposite of a modelling approach, because modelling implies that you are abstracting or not dealing
>> with the actual data, but in DOD we do the opposite, we focus on the actual data, to such a degree that we redefine its
>> actual layout to serve the transformation.
>>
>> DOD is, in essence, anti-abstraction (and therefore not-modelling).
>>
>> In practice, we find a balance between the anti-abstraction of pure DOD and code architecture component needs.

## What modelling approach should I follow, then?

I don't know. I think different people will find that different modelling approaches work better or worse for them, and
that is alright.

Personally, I just like to keep things simple, really simple, and build complexity only when 100% warranted. I've seen
only too many clean up, refactor, simplify, and address technical debt tasks to understand that complex solutions in the
name of some abstract goal (e.g. extensible, single responsibility principle, DRY, etc...) don't work out in the end.

So, what does simple mean to me?

### Simplicity

Simple means straightforward. Simple code is code that is easy to understand by itself, no knowledge of other code or
foreign concepts needed.

This immediately rules out most of the C++ STL, like:

```cpp
std::vector<int> v; // initialized somehow
int sum = std::accumulate(v.begin(), v.end(), 0);
```

This is not simple code, it's just short. But short doesn't make it simple. To understand this 2 line snippet you must
be familiar with a lot of concepts: `std::vector` (more complex than arrays), iterators, STL algorithms, templates or
generic metaprogamming, etc...

On the other hand, this is a lot easier to understand:

```cpp
int* values; // initialized somehow
int valueCount;

int sum = 0;
for (int i = 0; i < valueCount; ++i) {
  sum += values[valueCount];
}
```

This last snippet only requires basic programming concepts to be completely understood: variables, pointers, loops,
etc...

### Warranted complexity

There's only so much that can be done with simple loops and accumulating values. With growing amounts of features and,
therefore code, complexity will invariably materialize. However, striving for simplicity should always be on the front
of your mind, figthing [entropy](https://marccgk.github.io/blog/on-c-and-object-oriented-programming#on-entropy-as-the-hidden-force-behind-software-development)
and not giving in to the path of least resistance.

There are inherent complex problems, though, and they require complex solutions. One of such complex problems is
extensibility, that is a way for the user to provide custom functionality to some software without modifying the source
code. This problem justifies, and necessitates, a complex solution, e.g. a plugin architecture.

## How do you translate that into a modelling approach?

As I said, this can adopt different forms for different people. I can only speak for myself, but what follows is what
I've found to work for me and what I've learned from other people that take a similar approach to code.

### Write straightforward code

Code is easier to understand when it's written linearly, in a procedural way. That generally takes the form of long
functions that do one thing conceptually, but might be composed of multiple sub-tasks to accomplish the main one. These
sub-tasks are not extracted into their own standalone functions, though, until there's a reason for it, that is, the
code is already duplicated in 2 or 3 places and the commonality is large enough (i.e. 90%+) that the cognitive overhead
of another function is superseeded by its usefulness.

However, there's also room for small functions. In contrast with large functions that implement main features, short
functions tend to be helper functions that get called from the large ones and return immediately. Some examples of such
functions are allocations (e.g. a temporary allocator might just bump a pointer), make/create functions, math libraries
(e.g. vectors, matrices), etc...

Physically separating the code like this has several advantages both for the programmer and the compiler:

- Context: code is rarely useful in isolation, context matters a lot. Having code close together provides more context
    than separating it through multiple functions.
- Shallow call stacks: deep call stacks are a symptom of complexity. "Vertical implementations" (i.e. functions that
    call other functions) keep code away from useful context, making understanding harder.
- Compiler optimizations: keeping *secondary* functions short and shallow helps the compiler with better optimization
    opportunities, like inlining.
- Changes: changing code laid out sequentially in a large function is easier than having to modify multiple functions,
    parameters, return types, etc... It also makes it easier to have side by side implementations by reimplementing the
    entire function and adding a switch one level above.

### Solve the problems backwards

The purpose of all code is to transform data from one format to another (from [Mike Acton's Three Big
Lies](https://cellperformance.beyond3d.com/articles/2008/03/three-big-lies.html)). Therefore, you _know_ what the result
of the transformation looks like. Working from the final format backwards, makes it easier to write the least amount of
code to satisfy the transformation at every step.

For example, if you were to write a software rasterizer, you might want to start by allocating pixel memory for your
window and clearing to a single pixel color. Then, change some areas of the image to the color you want, e.g. the
corners. Then rasterize rectangles that are 100% within image bounds, later some rectangles that are not within bounds
(clipping). After that, try rasterizing triangles. And only after rasterization in 2D works, start working on 3D
geometry and projections, until you can load arbitrary 3D models and fully rasterize them.

This method will ensure you only write the solution to your immediate problem. The code will change a lot, but it will
change as the problem itself evolves. The patterns and algorithms will emerge from the solution and not from some
preconceived notion of what the solution should've been.

### Simple memory allocation models

Memory management tends to get a very bad rap for being too hard for mere mortals, and programmers are recommended to
stay away from it. That's why one of the main features of high-level languages is automatic memory management by e.g.
[garbage collection](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)), [automatic reference
counting](https://en.wikipedia.org/wiki/Automatic_Reference_Counting) or [smart
pointers](https://en.wikipedia.org/wiki/Smart_pointer).

Keeping in line with the simplicity principle, most memory allocations fall into simple models with just a small
percentage being hard problems.

One such simple model are temporary allocations. A lot of algorithms need some sort of scratch memory to save
intermediate results (e.g. [merge sort](https://en.wikipedia.org/wiki/Merge_sort)). A temporary allocator is all that's
needed for these use cases:

```cpp
struct Temp_Allocator {
  void *memory;
  U64 used, size;
};

struct Temp_Mem_Block {
  Temp_Allocator *allocator;
  U64 saved_used;
};

void *alloc(Temp_Allocator* allocator, U64 size, U64 alignment) {
  U8 *mem = ((U8 *)allocator->memory) + allocator->used;
  U8 *aligned = (U8 *)((((U64)mem) + alignment - 1) & ~(alignment - 1));
  U64 offset = aligned - mem;
  size += offset;

  assert(allocator->used + size <= size);
  allocator->used += size;

  return aligned;
}

Temp_Mem_Block begin_temp_alloc(Temp_Allocator* allocator) {
  struct Temp_Mem_Block result = { allocator, saved_used = allocator->used };
  return result;
}

void end_temp_alloc(Temp_Mem_Block block) {
  mark->allocator->used = block.saved_used;
}

struct Temp_Allocator g_tmp;
void some_function(...) {
  struct Temp_Mem_Block tmp_mem = begin_temp_alloc(&g_tmp);
  // do work
  end_temp_alloc(tmp_mem);
}
```

Another very common use case is program state, for which a growing memory arena that allocates memory blocks would
suffice. There are many different memory allocation strategies that can be applied to different problems but still, most
allocations will fall into these simple patterns.

My advice would be to avoid `new`/`delete` and `malloc`/`free` for most allocations, and reserve them to allocate big
memory blocks for the allocators/arenas to use.

The fact that memory management can be so much simpler than it is made up to be means that complex memory management
strategies like smart pointers and garbage collection are rarely needed.

#### Avoid RAII

[Resource Acquisition Is Initialization](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization) is
widespread in most object-oriented code. It theoretically provides a lot of benefits to its users, but it also comes
with a lot of complexity that spreads through the codebase.

Using RAII to acquire resources in class constructors and release them in class destructors in C++, forces another set
of behaviors on the programmer on everything that interacts with that code:

- Allocation/Initialization coupling: it's a lot harder to implement simple allocators like the one I've shown above,
    since every single allocation will need to call a constructor. Similarly, deallocations will need to call
    destructors. This prevents allocators from wiping out memory sections by resetting a pointer or clearing memory by
    calling `memset`. Additionally, allocation code will have to find ways to propagate parameters to constructors, so
    allocations are not restricted to default constructors.
- RAII cascade: classes that include other classes that use RAII are forced into RAII by extension.
- Resource lifetime linked to class lifetime: moving classes that implement RAII around gets a lot more complicated:
    custom code in copy constructors, move semantics, reference counting smart pointers are some of the _solutions_
    added to get around the added complexity.

### Inheritance

I've found that there's seldom need for inheritance the OOP way. There are 2 different simple solutions to inheritance
that make the code easier to work with in several ways.

#### Discriminated unions

```cpp
enum Command_Type {
  Command_Type_Add,
  Command_Type_Remove,
  // ...
};

struct Command_Add {
  // Add command data
}

struct Command_Remove {
  // Remove command data
}

struct Command {
  enum Command_Type type;
  // more common data

  union {
    Command_Add add;
    Command_Remove remove;
    // ...
  };
};
```

Some of the advantages of this method include:

- All objects have the same size, so they can be stored in a plain array
- All sub-types are explicitly stated and visible: no hidden variables, harder to duplicate data by mistake
- There's no hidden virtual table pointer

#### Composition

Some times, though, it's easier to use plain composition, e.g. different classes which have an explicit header as their
first data member.

```cpp
enum Command_Type {
  Command_Type_Add,
  Command_Type_Remove,
  // ...
};

struct Command_Header {
  enum Command_Type type;
  // more common data
};

struct Command_Add {
  struct Command_Header header;
  // Add command data
}

struct Command_Remove {
  struct Command_Header header;
  // Remove command data
}
```

The disatvantage of this approach is that it's no longer possible to declare a plain array of `Command`s, so one needs
to push the different commands to a raw memory buffer and later, when reading, grab the header, figure out the type and
skip `sizeof(Command_XYZ)` to get to the next one. However, this works better when certain types need to be accompanied
by a varying amount of data, e.g. a buffer of transforms, since the data can be stored immediately after the type itself
and how many bytes to jump to the next type can be inferred from the data itself.

The trick here, is that because how memory layout works in C, we know that `header` in `Command_Add` will be at an
offset of 0 bytes from the start of the `struct`:

```cpp
U8 *cmd_mem; // filled somehow

Command_Header *header = cmd_mem;
while (header < /* array end */) {
  switch (header->type) {
    case Command_Type_Add: {
      Command_Add *cmd = (Command_Add *)header;
      header = (Command_Header *)(((U8 *)header) + sizeof(Command_Add)); // skip Command_Add
      // work with Command_Add
      break;
    }
    // more cases ...
  }
}
```

### Arrays & Loops

Probably, the most common operation in software is to process lists of things. Keep data that is processed similarly
together, i.e. in the same array, so critical code can have this form:

```cpp
void transform(F32 *out, F32 *in, U64 count) {
  for (U64 idx = 0; idx < count; ++idx) {
    out[idx] = in[idx] * 3.f + 2.f;
  }
}
```

### Multithreading

Multithreading is hard. As a general rule, I try to divide work in tasks that can operate as small single threaded
programs, so the same function works in both single threaded and multithreaded cases. I also like keeping some data per
thread, like a temporary memory allocator, which can be aggregated in a per-thread context.

```cpp
struct Thread_Context {
  struct Temp_Allocator allocator;
  // other
};

void process_task(struct Thread_Context* ctx, U32* input, U32* output, U32 count) {
  struct Temp_Mem_Block tmp_mem = begin_temp_alloc(&ctx->allocator);
  U32* tmp = ctx->allocator.push_array(U32, count);
  sort_U32(input, tmp); // sort input using tmp as scratch memory

  // do some work
  for (U32 i = 0; i < count; ++i) {
    output = operate(input);
  }

  end_temp_alloc(tmp_mem);
}

// The job scheduler has a Thread_Context per thread, that passes to the task function
```

A more comprehensive view on multithreading is out of this article's scope.

### Be explicit

If there's a common theme to all the ideas I've written about in this post it is **be explicit**. Being explicit is
always preferable to being implicit.

I'm definitely not alone in making explicitness a priority, e.g. [Our Machinery](https://ourmachinery.com)'s Guidebook
[Design Principles](https://ourmachinery.com/files/guidebook.md.html#omg-design:designprinciples) are also outlined
along this same principle.

### C++ and Language Features

All the code in this post, and [previous](https://marccgk.github.io/blog/a-virtual-memory-linear-arena/)
[posts](https://marccgk.github.io/blog/optimizing-the-cas-loop/) is written in C, not in C++. The main reason is that C
is a simpler language than C++, so I can focus on the basic building blocks.

Specially for beginners, I think it's worth understanding the basic constructs, before jumping head first into language
features that might derail your goals because of unforseen trade-offs.

## Closing

What I wrote in this post are guidelines for some modelling approaches that will generate code somewhat following a
Data-Oriented Design. There is nothing new or revolutionary in this post, but I wanted to gather some thoughts and
advice I've followed myself in a single place, for future reference. Take this as my drop in the ocean to try and make
the path to better software easier for anyone who wants to try.

For a list of resources with similar ideas and more, please take a look at the end of [my last post](https://marccgk.github.io/blog/on-c-and-object-oriented-programming#silver-lining).

[^1]: Not an empirical study, based on my experience working on C++ codebases.
