---
# vim: set textwidth=120:

title: "On C++ and Object Oriented Programming"
date: 2019-01-15T11:05:09-08:00
draft: false
---

Much has been [written](http://aras-p.info/blog/2018/12/28/Modern-C-Lamentations/) lately about
[C++](https://mikelui.io/2019/01/03/seriously-bonkers.html), the direction the language is taking and how most of what
gets called "modern C++" is just a no-go zone for game developers.

Although I fully agree with the sentiment, I tend to look at C++ evolution as the effect of a pervasive set of ideas
that dominate the minds of most developers. In this post I'll try to put some of my thoughts on these ideas in order
and, hopefully, something coherent will come up.

## On Object Oriented Programming (OOP) as a tool

Even though C++ is described as a [multi-paradigm programming language](https://en.wikipedia.org/wiki/C%2B%2B), the
truth is that most programmers will use C++ exclusively as an Object Oriented language (generic programming will be
used to "augment" OOP). Even beyond C++, newer languages have been invented implementing Object Oriented Programming as
a first class citizen with more features that the ones present in C++ (e.g.
[C#](https://en.wikipedia.org/wiki/C_Sharp_(programming_language)),
[Java](https://en.wikipedia.org/wiki/Java_(programming_language))).

OOP is supposed to be a tool, one of multiple paradigms that programmers can use to solve problems by writing code.
However, in my experience, OOP is taken as the *gold standard* for software development by the majority of
professionals. As such, developing solutions starts by deciding which objects are needed. The actual problem solving
starts *after* there's an object or objects that will host the code. When that's the thought process, OOP is not a tool,
but the entire toolset.

## On Entropy as the hidden force behind software development

The way I visualize an OOP solution is like a constellation of stars: a group of objects with arbitrary links between
them. This is not different from looking at it as a graph, where objects are the nodes and the relationships, the edges,
but I like the notion of group/cluster that constellation conveys (vs the abstract meaning of graph).

What worries me, though, is how this "constellation of objects" happens to be. As I see it, these constellations are
nothing more than a snapshot of the programmer's mental image of what the solution space is at a given time.
Even taking into account all the promises OOP design makes about extensibility, reusability, encapsulation, etc...
no one can see the future, so the only possible solution one can implement is the solution to the immediate problem one
has in front.

The fact that we're "just" solving the immediate problem at hand should be good news but, in my experience, when using OOP
design principles, programmers write a solution while shackling themselves to a commitment that the problem won't change
significantly and, therefore, the solution is permanent. I mean, from now on the solution is to be talked about in terms
of the objects that form the constellation instead of e.g. data and algorithms; the problem has been abstracted away.

However, software is subjected to entropy as much as any other system and, therefore, we all know the code will
change. And we all know the code will change in unpredictable ways. What is very clear to me, though, is that code will
always degrade into chaos and disorder unless consciously fought against.

I've seen this take many forms in OOP solutions:

- New intermediate levels in a hierarchy appear where there were supposed to be none.
- New virtual functions are added, that have empty implementations in most of the hierarchy.
- One of the objects in the constellation takes more work than it used to, causing friction in the links between
    other objects.
- Callbacks are added in the hierarchy so objects in one level can talk to objects in a different level without
    explicit knowledge of each other.
- etc...

These are all examples of extensibility gone wrong. And it invariably leads to the same situation, either a few months
down the line or a few years. **Refactoring** will try to correct the violations of OOP design principles
added to the constellation of objects by changes in the problem. Sometimes, it will. For a while. Entropy doesn't stop,
and programmers have limited time to refactor each OOP constellation to fight it, ending in the same situation without
fail: chaos.

There's a point in the lifetime of an OOP design where it has degraded to an untenable situation. There are mainly two
actions that can be taken at this point:

- Black Box: hide the constellation behind some kind of facade and slowly pull it away from the rest of the code. The
    system can continue solving the original problem if it still performs well enough, but feature development has
    completely stopped and bug fixes take very long, if successful at all.
- Rewrite from scratch: the OOP design used to solve the original problem is so far away from the current state of the
    problem, that no amount of incremental refactoring is capable of adapting the current solution.

Note that a black box will need a rewrite in case further feature development and/or bug fixing is still needed.

Rewriting the solution takes us back to the notion of a snapshot of the current solution space. So, what changed between
when OOP design #1 was written and now? Everything, actually. The problem changed, hence the solution needs to be
different.

By writing the solution following OOP design principles, we abstracted the problem away and, as soon as it changed,
the solution fell apart like a house of cards.

This is where I would expect people to wonder what went wrong, try a different path and update problem solving
strategies based on the *postmortem* results. However, every time I've seen this "it's time to rewrite" scenario,
the same strategy is used: OOP design principles, coding a snapshot of the *new* mental image of the new current problem
space. And so the cycle repeats itself.

## On easy to delete code as a design principle

In any OOP based system, solutions are implemented as a "constellation of objects", making the objects the centers of
focus. However, I think the relationships between objects are as important, if not more, than the objects themselves.

I prefer simple solutions, that add the minimum amount of nodes and edges to the dependency graph of the code. The
simpler the solution, the easier is not only to change, but to delete. And I've found that the easier code is to delete,
the faster you're able to turn solutions around and adapt to changing problems. At the same time, the code becomes more
resilient to entropy, since it takes a lot less effort to keep it in order and avoid the descend into chaos.

## On performance by design

One of the main reasons to avoid OOP design, though is performance. The more code you have to run, the worse performance
will get.

There's also the fact that OOP features are inherently poor performance-wise. I implemented a simple OOP hierarchy with
an interface and two derived classes that override a single pure virtual function call in
[Compiler Explorer](https://gcc.godbolt.org/z/1nftf-).

This example code either prints "Hello, World!" or not, depending on the number of arguments passed to the program.
Instead of directly coding what I just described, however, the code will use a standard OOP design pattern, inheritance,
to solve the same problem.

What stands out the most is how much code is generated by the compilers even after optimizations. Then, looking more
closely, you can observe how expensive it is to do nothing: when the number of arguments to the program is not zero, the
code will still allocate memory (call `new`), load the `vtable` addresses for both objects, load the `Work()` function
address for `ImplB` and jump to it to return immediately since there's nothing to do. Finally, call `delete` to free the
allocated memory.

None of these operations where necessary at all, but the processor dutifully executed them regardless.

Now, if good performance is one of your product goals (why wouldn't it?) then code needs to avoid unnecessary costly
operations and favor simpler, easy to reason about constructs that help reach that goal.

Take [Unity](https://unity3d.com/), for example. Their recent **performance is correctness** effort uses C#, an object
oriented language, since it's the language already in use in the engine. However, they chose to use a
[subset of C#](http://lucasmeijer.com/posts/cpp_unity/), namely the non-OOP subset and build performance aware
constructs on top of it.

Given that the job of a programmer is to solve problems using computers, it's mind-boggling that so much of our industry
cares so little about writing code that actually takes advantage of what CPUs are good at.

## On changing minds

[Angelo Pesce's](https://twitter.com/kenpex) blog post
[Over-engineering (the root of all evil)](https://c0de517e.blogspot.com/2016/10/over-engineering-root-of-all-evil.html)
hits the nail on the head (see last section: **People**) on recognizing that most software problems are actually people
problems.

Teams need to work together and have a shared vision on what's the goal and the path to get there. If people on a team
disagree on, e.g. the path to reach the goal, some consensus needs to be attained in order to keep moving forward. This
is generally not a big deal when differences in opinion are small, but it becomes a big issue when the options differ at
a fundamental level, e.g. OOP vs no OOP.

I learned to program in Java and was taught Object-Oriented Programming principles as part of the Fundamentals of
Software Development. OOP Design Patters followed soon after. I later learned C++, which I've used ever since. As I
acquired professional experience my comfort with C++ increased: I was able to work with larger and larger codebases and
keep more concepts in my head at once, I could also write more complicated code using advanced language features.

However, I always had this nagging feeling that something wasn't right. I kept feeling frustrated while working with
class hierarchies, the syntax, compiler error messages and various shenanigans of template metaprogramming annoyed me to
no end, I had no patience for absurdly long compile times (still don't) and overly complicated solutions to simple
problems irritated me even though it was "proper OOP design".

That's why when I learned about alternatives, mainly that OOP is a design **decision** not a **requirement**, I started
exploring different paths.

Changing your own mind is hard. Challenging your own opinions, realizing how wrong you were and correcting course is
difficult and painful. However, it's nowhere near as hard as changing someone else's opinion!

I've had a lot of conversations about OOP and its inherent problems with different people and, although I think I've
managed to convey why I think the way I do, I am not sure I've swayed anyone towards the non-OOP way. Maybe this post
will help.

Over the years, though, I've seen three main arguments that prevent people from giving the other side a chance:

- "Good OOP wouldn't do this.", "This is badly designed OOP.", "This code doesn't follow OOP principles." and similar
    variants. I've heard this one when showing examples of OOP gone bad (I've already talked about why OOP code
    [invariably goes wrong](#on-entropy-as-the-hidden-force-behind-software-development)). This is a
    clear case of the [No true Scotsman fallacy](https://en.wikipedia.org/wiki/No_true_Scotsman).

- "I know OOP, if I have to start from scratch, I won't have anything anymore". This is fear to loose one's seniority
    if after a career of using OOP principles and leading other people on these principles, they had to start developing
    coding skills with a completely different mindset. I believe this is a case of the
    [sunk cost fallacy](https://en.wikipedia.org/wiki/Sunk_cost#Loss_aversion_and_the_sunk_cost_fallacy).

- "Everyone knows OOP, and it's very powerful to have the shared knowledge for communication". This is an [appeal to the
    majority fallacy](https://en.wikipedia.org/wiki/Argumentum_ad_populum), i.e. if virtually every software developer
    uses OOP principles, the idea can't be wrong.

I'm fully aware that identifying arguments as fallacies is **not** suficient to disprove their validity. However, I do
believe that being aware of your own lapses in judgement can help you dig deeper and find the root cause of your
rejection of a different idea.

To summarize: engineering problems are easy to solve, but people problems are really hard!

## Silver lining

There are lots of people that take performance and quality software crafting seriously and they're vocal about it. I
learned a lot reading blogs and watching talks that challenged my views and forced me to think deeply about opinions
that are considered established knowledge.

I've compiled a non-exhaustive list here:

[Mike Acton's](https://twitter.com/mike_acton) talks at
[CppCon 2014: Data-Oriented Design](https://www.youtube.com/watch?v=rX0ItVEVjHc) and at
[GDC 2015: Code Clinic](https://gdcvault.com/play/1021866/Code-Clinic-2015-How-to).

[Casey Muratori's](https://twitter.com/cmuratori) blog posts on how to program:

- [Semantic Compression](https://caseymuratori.com/blog_0015)
- [Complexity and Granularity](https://caseymuratori.com/blog_0016)
- [Designing and Evaluating Reusable Components](https://caseymuratori.com/blog_0024)

And, of course, his excellent [Handmade Hero](https://handmadehero.org/) game development project.

[Christer Ericson's](https://twitter.com/ChristerEricson)
[design patterns are from hell](http://realtimecollisiondetection.net/blog/?p=44) (and the
[follow up](http://realtimecollisiondetection.net/blog/?p=81)).
[This other post](http://realtimecollisiondetection.net/blog/?p=80) contains some links about memory management and
optimization that, although old (circa 2008!) are still fully relevant today.

[Sebastian Aaltonen's](https://twitter.com/SebAaltonen) "controversial" thoughts on
[Twitter](https://twitter.com/SebAaltonen/status/1080069784644059139), which I happen to fully agree with.
