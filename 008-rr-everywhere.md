_21 March 2025_

### [Other writings](README.md)

# Bringing Record and Replay everywhere

## TL; DR

I've modified the awesome [`rr`
debugger](https://github.com/rr-debugger/rr.git) so that it can run
without needing access to CPU Hardware Performance counters.

This allows `rr` to be used in many more environments like cloud
VMs and containers where access to CPU HW performance counters is
usually disabled. Upstream `rr` requires access to CPU HW performance
counters to function, so this is a new feature and opens many
additional possibilities to debug your software with `rr`.

I call this variant, _Software Counters mode_ `rr`. Record and
Replay systems like `rr` deserve to be able to run everywhere and this
is my attempt at making this possible !

To build, install and run _Software Counters mode_ rr please visit
https://github.com/sidkshatriya/rr.soft

_Continue reading to understand some of the **basic concepts**
behind Record and Replay and why it is such a powerful technique
when debugging programs._

## What is program record and replay ? Why is it useful ?

When you watch a YouTube video, there is the unspoken expectation
that when you rewind or go forward an arbitrary number of seconds the
video is exactly the same as it had been when it was recorded !

You don't see something new in the video that was never there ! The
video and audio is exactly the same. In fact, if the video was
slightly different when you replayed it again and again
it would be extremely strange. You would probably think you were
going mad if the video were slightly different on each replay :-)
!

Allowing you to go forwards and backwards in time when viewing a
video is critical. Once a video is uploaded to the YouTube website,
it is essentially "frozen". This allows anybody to view the same
video any number of times and concentrate or learn from parts of
the video that are most interesting.

### From replaying videos to programs

Let us shift focus from YouTube videos to computer programs.

Let's say you're attempting to run a program again and again because
you are facing some errors. When you try to debug an error, a good
strategy is try to give the program the same input because you want
to understand why it is failing for your specific input.

But sadly, every single time you run a program things are slightly
different:

- Your user input like keystrokes may be the same but the timing of these
  keystrokes can be subtly different. This could result in slightly different
  internal state of a TUI (Text User Interface) program from run to run even though
  the raw text you may have entered is the same
- If the program has a graphical IDE, even though you may click on the same buttons, the
  mouse speed or mouse path is slightly different from the last run or exact mouse click
  locations differ by a few pixels
- If the program makes network calls, remote servers might respond to requests
  slightly differently from run to run -- there might be network failures, or
  because the state of the remote servers is different they might still respond differently
- If the program depends on time or random numbers the program might run a bit differently
  as the time has changed or different random numbers are generated on every run
- If the program depends on files on disk, the files may have gotten modified the last time the program
  ran and as a result the error may not appear again. It may be difficult to restore the
  files to the state in which the same error in the program happens again.
- If the program is multithreaded, the different threads might interleave in slightly
  differently ways from run to run so the results might be different

**I hope you get the picture: every run of a sufficiently complex program is a special "Snowflake"
even though you may try to give it the same input.**

So when a program goes wrong you have not one but two simultaneous
problems:
1. Find where the program bug is in the code
2. Try to set the conditions of your system and replicate
user input (and often subtle things like thread interleavings) in such a way that the program
when run again shows the same bug.

But as discussed above, so many things can change from run to run
! In fact, this is the age old problem of engineers "But it works
for me!" or alternatively "There was horrible error, likely to
appear in production... but I don't know how to get the error again
!"

### Record and Replay to the rescue

What if while running the program you were simultaneously recording
it using a record/replay facility ? Think of a video camera of
sorts, but for programs. So that when you replay it back, it runs
in exactly the same way: the CPU instructions executed are exactly
the same, when files are "read" or "written" or when network calls
are made, the results are exactly the same and so on and so forth.

As another example, say you recorded a vim/emacs session using this
record/replay facility. When replaying the recording, the program
code thinks that you've pressed exactly the same keys in the same
exact time sequence.  The internal state of vim/emacs will be exactly
the same !  Now if there was a crash in vim/emacs that occured
during the record phase you'd be able to review it again (by replaying
again) and drill down to the functions that were executing and
inspect the program state in the data structures.

How does this all work ? During the recording phase, all non-deterministic
aspects of program execution like random numbers, user inputs,
system calls for the clock time and other system calls results are
saved to a log. Then while replaying the program back, those
"non-deterministic" aspects of program execution are inserted as
necessary in the replayed program.  To be more accurate, when the
program is replayed many things don't actually happen again; for
example actual network calls are not issued again. The replayer
simply consults the log for the stored network call results and
inserts those network call results.  The replayed program thinks
it made a network call but actually it didn't. You could say all
sources of non-determinism are inserted automatically during replay
and the program is none the wiser.

### What do we gain ?

Now if we have the ability to record and then faithfully replay a
program "tree" (a program may start another program which may start
other programs and so on. This is an execution "tree") then it
becomes an extremely powerful feature. **If you capture a bug during
record, that bug will appear during every future replay of that
recording**.

Record and replay is useful even if you don't want to fix any bugs
in your program.  It may just be that you want to understand how
your program works for a specific input. Now that program behavior
does not change for a specific recording, you can review that run
of the program any number of times and zoom into that program replay:
see how each function calls another function and so on. **Think of
this as reviewing a tricky part of a video lecture multiple times
to really understand what the lecture is teaching**.

Additionally if you have the record/replay facility, you could do a
lot of amazing things like unleash a gbd/lldb debugger on a replay.
You could step forward/backward into the execution, examine
backtraces, variables, the sky is the limit.  And the best part is
that once you replay, you are replaying a "frozen" run of the
program. It behaves the same everytime you replay/debug it, just like our
YouTube video !

## `rr`, an open source Record and Replay debugger

There are [many](https://github.com/rr-debugger/rr/wiki/Related-work)
systems that can perform record/replay. Many of the systems are
either proprietary, abandoned or have various constraints that make
them pratically unusable. As of this writing I am aware of only 1
open source record/replay system for Linux which has all the following features:
1. Broad based usability (records/replays programs written in any language)
2. Reasonably good performance
3. Is robust (e.g. handles signals well which is the bane for many a record/replay system)
4. Works in a sufficiently non-instrusive way (does not need a custom kernel module, a custom libc, root privileges etc.)
5. Is minimalistic (e.g. only records program "tree" of interest and not the whole virtual machine)
6. Works with off-the-shelf debuggers like gdb and lldb
7. Is being continuously improved and developed
8. Is widely used in industry

And that system as you would have guessed, is [rr](https://github.com/rr-debugger/rr).

BTW the best resource to understand how record and replay systems
in general and `rr` in particular works is to read the arXiv paper
[Engineering Record And Replay For Deployability: Extended Technical
Report](https://arxiv.org/abs/1705.05937). Note that the report was
written in 2017 and even though there have been a lot of changes to the
code (more robustness, lots of bug fixes, more features, aarch64 support etc.) 
the core concepts behind `rr` remain the same.  In my opinion
it is still the best resource to learn about `rr` at a slightly
deeper level. You should brush up on topics like Linux system calls,
signals, ptrace etc. before diving into the paper (there are a lot
of great books on Linux/Unix systems programming -- choose any one).

### A limitation of `rr`

Now every program makes a LOT of systems calls related to user
input, network calls, memory layout, threads, processes etc. that
are non-determinisitic (i.e.  can subtly change from run to run).
When a program is being replayed the record/replay system needs to
consult that log (talked about above).  But how can we know which
entry of the log to consult ? How do we know how far the program
progressed ? We could simply count the system calls made so far as
a measure of progress but that does not work for many reasons. Here
is one reason: while recording a multithreaded program, threads are
often scheduled and descheduled (prempted) on the CPU. The premption
points are arbitrary i.e. often can happen even when the program
is _not_ invoking a syscall. How do we know *exactly* when to switch
threads during replay so that our program runs with exactly the
same thread interleaving during record ? This is where HW performance
counters can be useful. An example of a HW performance counter could
be number of assembly branches executed since the start of the
process.  These counters (access available through the Linux perf
API) can help measure program "progress". They allow you to see how
far a program has progressed _exactly_ so you know what record to
consult in the log so that it can be consulted during replay and
do things like decide to switch threads. This is a highly simplified
explanation.

While `rr` is a fantastic system, it has one big flaw: It assumes
that programs can access the "CPU Hardware Performance Counters"
discussed above.  These performance counters are rarely enabled in
cloud VMs and containers as discussed.

## Interlude: My love affair with Record/Replay and `rr`

For many years now, I've been fascinated with systems that do record
and replay specifically `rr`. I've made some code contributions to `rr`.

`rr` is unfortuntely built in (mostly) C++ which is in my opinion a
large, bloated and complex language. Much as I respect the scalability
and bare metal nature of the C/C++ language I think better alternatives
like Rust exist.

I'd like to specifically give a shout out to an old project of mine,
[`rd`](https://github.com/sidkshatriya/rd.git) - a port of `rr` to
Rust.

I wrote 4 blog posts on `rd`. Read the last blog post on rd
[here](004-A-debugger-is-born.md).  It has links to previous blogs
on `rd` should you be interested to read all of them.

A brilliant team continues to work on `rr` making bug fixes and
adding new features. It just makes sense to use `rr`. It was not
fun for me to keep porting things back to `rd`. I decided to retire
(archive) the `rd` project for this and various other reasons,
sadly.

Now the experiment in porting a large code base from C/C++ to Rust was
a fantastic learning experience nevertheless and allowed me to
understand the highly complex codebase of the `rr` debugger.

## The need for _Software Counters mode_ `rr`

Over the many years in this space I realized that people are working
more and more in restricted or constrained environments like cloud
VMs and containers where CPU HW performance counter access is usually
disabled. I wanted to bring the joy and power of `rr` to these
environments.

The result is _Software Counter mode_ `rr` !

**Running rr record/replay without access to CPU HW performance
counters is accomplished using lightweight dynamic (and static)
instrumentation**. The _Software Counters mode_ rr
[wiki](https://github.com/sidkshatriya/rr.soft/wiki) has more details
in case you're curious about some more of the internals.

Building _Software Counters mode_ `rr` has truly been a labour of
love for me.  It's been a hugely rewarding journey for me as I
embarked even deeper into so many areas relating to dynamic and
static instrumentation, Linux system internals, signals, low level
assembly, DWARF/ELF, and much more.

Please see https://github.com/sidkshatriya/rr.soft to learn how to
build and run _Software Counters mode_ `rr` on your own machine.

---

_As this is a personal blog, I don't accept any Pull Requests to
it currently. If you wish to bring something important to my attention
regarding this or any other blog post, you can file an issue. Thanks
for reading!_
