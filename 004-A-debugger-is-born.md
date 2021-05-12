_12 May 2021_

### [Other writings](README.md)

#  55,000+ lines of Rust code later: A debugger is born!

I am proud to announce that the [Record & Debug Tool (`rd`)](https://github.com/sidkshatriya/rd) has entered alpha.

The Record & Debug Tool ([`rd`](https://github.com/sidkshatriya/rd)) is a record/replay debugger written in Rust. It is a project to port the [rr](https://github.com/rr-debugger/rr) project from C/C++ to Rust. Why is this a good idea? Why Rust? For that, please read some of my older posts and updates:

- [12 Aug 2020 - Introducing rd: A port of mozilla/rr to the Rust programming language](001-rd-intro.md)
- [13 Aug 2020 - rd: Why I chose Rust instead of Golang, OCaml or Dlang for the mozilla/rr debugger port](002-why-rust.md)
- [28 Dec 2020 - 45,000+ lines of Rust code later: An update on the Record & Debug Tool (`rd`)](004-A-debugger-is-born.md)

**Current Status**: The port is substantially complete and is currently of an alpha level quality. You should be able to use `rd` for the tasks you would ordinarily use `rr` for. The `rr` project keeps accumulating features and fixes and many of them have not found their way into rd yet. However, my expectation is that `rd` should be reasonably robust, complete and usable now. Please try it out!

Today in this blog post I want to go deeper into why debuggers matter and why you should care about the Record & Debug Tool ([`rd`](https://github.com/sidkshatriya/rd)).

## Why do debuggers matter?

In my opinion, debuggers are under-appreciated and under-invested tools in a programmer's arsenal. Now, there are hundreds of new programming languages around with new ones appearing every year. There are also an uncountable number of articles that appear regularly on technical blogs about trying out a new programming language or gushy posts about a new feature introduced in this year's hot programming language!

The same cannot be said about debuggers: the field of debuggers is decidedly more sedate. New debuggers arise on the scene very occasionally. Most people could probably only name `gdb` and `lldb` as the general purpose debuggers they are aware of. You will also rarely see fans wax eloquent about new features in a new release of gdb/lldb (there are always exceptions though). 

Yes, there are language specific debuggers for Golang, Python, PHP, Java, .Net etc. but even these debuggers hardly receive the attention in proportion to the value they provide, and the sheer time end-users spend using them.

There are various historical and practical reasons for this phenomenon of under-appreciation and under-investment. Debuggers as they are currently implemented are often not very useful for various reasons -- bugs are often difficult to repeat, once the bug is repeated, it is tough to pin down (have you ever stepped too far in the debugger only to have to restart your 10-min setup again?), the interface is too low level, the breakpoint paradigm has limitations etc. In fact there often many capable developers who will insist that debuggers are not useful and will avoid them at all costs -- they will encourage other approaches like program logs etc.

Furthermore, debuggers are seen as unsexy, messy, special case ridden pieces of engineering that are pointlessly hard to build and lack the theoretical beauty that one sees in compiler design and implementation. The truth is somewhat more nuanced: Yes debuggers are tremendously messy and special case ridden. That is because they necessarily need to operate at a low level of abstraction as they often deal with registers, interrupts, memory etc. But then, compilers are _just as messy_ -- most people don't get to see that because they simply do the parsing, type checking and IR generation and leave the dirty parts to, say, LLVM.

Let's talk about the other complaint: debuggers don't have a theoretically interesting core. Nothing could be further from the truth. There is nothing inherently pedestrian about the work a debugger does. Just as compilers uses sophisticated algorithms during _compile-time_, a good debugger needs to employ sophisticated analyses and algorithms during _run-time_. Except the main problem is that very few debuggers actually do this today!

Debuggers have not experienced the revolution that compiler design and implementation has experienced over the last few decades. There are tons of academic papers written and Ph.Ds. minted every year in the field of compilers and PL theory every year. The same cannot be said for the field of debugger design and implementation. Debuggers, at least in the popular perception, have not significantly progressed in terms of capability over the last few decades.

This can only mean one thing: the field of debuggers is ripe for disruption and innovation. Just like `llvm` facilitated the rise of hundreds of new computer languages, I forsee a similar rise for debugging and debuggers in the coming years.

## What will be the LLVM for debuggers?

Science and engineering is based on the important principle of repeatability. You should be able to repeat something reliably in order to fix it, study it, improve it. Debuggers are plagued by this issue of lack of repeatability in the things they are studying: programs under execution. Bugs are ephemeral: sometimes they reveal themselves and sometimes they don't. So most debuggers like `gdb`/`lldb` can be quite unsatisfying tools because they can't reliably "home-in" on the bug they are trying to resolve. This is possibly one of the reasons why people don't invest so much in debuggers -- the current debugging paradigm is perceived as having inherent limits and it is just not powerful enough.

To resolve this problem of repeatability, record-replay debugging was invented quite some time ago. The concept is simple: record an execution of a program that exhibits a bug and then replay the same *exact* execution later. The bug will always appear in the replay and you can reliably debug the problem. There are many record and replay systems [around](https://github.com/rr-debugger/rr/wiki/Related-work).

However, for a long time there was a problem: most record-replay systems either were proprietary or limited in ability in some important way or both.

The [rr](https://github.com/rr-debugger/rr) debugger when it was released was a breath of fresh air. Finally, there was a viable open-source record-relay implementation! In the annals of debugging, I think this was a significant event: The `rr` debugger not only has novel technical aspects, it is quite "practical" and usable for day-to-day debugging needs.

Thinking beyond, record-and-replay debugging provides a platform on which you can super-charge debugging. By solving the repeatibility issue you can fix your bug more easily. But more importantly, you can now build some really amazing and powerful debugging tools on top of this "base debugger". You can do powerful analyses and take a data-mining approach to studying your program. As a further example, for people who are fans of logging, you can do something amazing: **retro-logging**. This allows you to insert some logging statements in the areas of the program you are interested in _after the program has already run!_ Because you have already recorded the execution, the program replayer can print out these logging statements when it replays the program.

The important take-way here is that `rr` provides a platform (just like LLVM did for compilers) which will allow the next generation of debugging tools to be built. (Companies like Pernosco are doing just that).

Of course, I want to point out that record-replay debugging in the `rr` paradigm is not a silver bullet. Parallel execution is essentially serialized during record. So a large class of concurrency issues cannot adequately be solved by `rr`. Recording/replaying programs executing parallely on multiple CPU cores reliably and in a light-weight way is still a research problem.

## A debugger written in Rust: `rd`

So where does `rd` come into the picture? A huge amount of compiler/debugger infrastructure is built in the C/C++ language (gcc, llvm, gdb, lldb, Swift, javac etc). The problems and complexity of C/C++ are well documented. Writing effective C/C++ is like walking a tightrope: a small mistake and you'll basically fall to your death. Modern languages like Rust provide extra guard-rails while not compromising on performance. They also don't assume that it's programmers are as smart as Herb Sutter! :-). Tough things like concurrency and parallelism in C/C++ become easier in Rust.

Sadly due to historical reasons, `rr` is also built in C/C++. Some of main developers of `rr` are huge Rust fans and I'm pretty sure if `rr` were built by them today, they would choose Rust as the implementation language. See this [interesting](https://github.com/rr-debugger/rr/issues/2181) discussion.

However, does that mean that like LLVM we are forever stuck with C/C++ for a foundational piece of debugging software like `rr`?

**No**. the `rd` port of `rr` to Rust is my attempt to change that inevitability. I've been obsessed by reverse-debugging for some time. I also believe that often the best way to learn something really well is to re-do it in some way. The `rd` port has been the expression of my love affair with the record-and-replay paradigm of debugging. I'm very happy that I've been able to come so far. Many times I thought I should give up -- I've documented my experiences while porting `rr` to Rust in the previous blogs that I linked above.

This port has been a long and lonely journey so far but I think `rd` has reached a stage that end-users can use it. Hopefully now that this project has reached this milestone, other developers can play around with it too and give feedback and contribute.

I have a few dreams and plans for `rd`. My hope is that in the future it can become a laboratory for some exiting features in debugging. But this will only come to fruition once `rd` becomes a reliable and much loved piece of software that `rr` already is.

Please play around with `rd` and let me know if things worked out for you. This is alpha level software, so I apologise in advance if things don't work as expected!
