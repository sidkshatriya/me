_13 August 2020_

### [Other writings](https://github.com/sidkshatriya/me/blob/master/README.md)

#  `rd`: Why I chose Rust instead of Golang, OCaml or Dlang for the mozilla/rr debugger port

My last post introducing [`rd`](https://github.com/sidkshatriya/rd), a Rust port of the [mozilla/rr](https://github.com/mozilla/rr) debugger got an enthusiastic [response](https://www.reddit.com/r/rust/comments/i8bmgq/rd_a_port_of_mozillarr_to_the_rust_programming/) on the reddit rust channel.

Today I'll like to talk more about why I chose Rust (instead of some other languages I was considering).

Please read my [previous post](https://github.com/sidkshatriya/me/blob/master/001-rd-intro.md) to get more context and background.

Before deciding to port rr to Rust I seriously considered using Golang, OCaml, dlang also. Why did I end up chosing Rust in the end? Here are the factors that entered my mind while making this decision...

_Note_: this post is opinionated and impressionistic in parts. You may not agree with some of the things I've said i.e. your mileage may vary. You've been warned :-) !

## Performance was _not_ an important consideration when choosing Rust

`rr`, `gdb` and most debuggers use the ptrace system call extensively to control the program being debugged. Now, the ptrace system call is probably one of the more complex systems calls in linux. Also, in general, system calls are very slow when compared to various other things a program does while executing. It's shocking how slow they are: you may have seen presentations that compare how much time it takes to perform an addition, access a register, L1 cache, main memory etc. system calls in comparison could take eons. 

Not only do debuggers need to use ptrace, they need to use ptrace _a lot_. So speeding things up is really about reducing the number of ptrace (and other system calls) and generally avoiding context switches between the debugee and the debugger.

`rr` has a lot of tricks up its sleeve to avoid issuing excessive ptrace calls and avoid unecessary context switches. In brief, it injects a small amount of code via `LD_PRELOAD` into the debugee program. This code orchestrates storing the results of various system calls (which are basically non-deterministic) into a buffer. When the buffer is full, rr is notified and the buffer is flushed and eventually stored to disk. There are a lot of complications e.g. not all system calls can be handled this way, rr must guard against deadlocks etc. But in general the whole process is quite amazing and effective. (More details can be found in a technical [report](https://arxiv.org/abs/1705.05937) about rr at ArXiv).

So in summary, it was _not_ really important to choose Rust for performance reasons. System calls like ptrace are really slow and we can't avoid _all_ of them. Therefore Golang and OCaml could have been used for rd also. A good amount of speedup comes from avoiding context switches between debugger and debuggee via the syscall buffering mechanism mentioned above which does not even run in the debugger.

In conclusion, I could have chosen golang, dlang, OCaml to port rr to. The bottlenecks for a system like rr are the system calls that need to be intercepted and then recorded during the recording phase and the bottleneck is (partially) resolved by tracee side code injection. This is somewhat akin to using Python, Ruby and PHP on your websites. Often the network latency + database access latency dominates and for many scenarios it does not matter if you used Rust or Python.

## Multithreading was _not_ a consideration while choosing Rust.

Broadly speaking the rr record/replay debugger is a single threaded program (there are some exceptions e.g. trace compression). Moreover, the tracee programs it is recording are themselves serialized. In other words, the tracees run concurrently but not parallely. 

So the Rust's super-powers in writing high performance multithreaded code was not really a big draw for the `rd` project. Also even things like Rust async were not really relevant for the rd project. It is nice to know they exist for doing cool things in the future but it's not hugely important right now.

Given all this, I _could_ have chosen OCaml to port `rr` too, if I really wanted. The lack of multicore runtime in OCaml therefore was not a big problem to me (OCaml might get a multicore runtime soon though). 

Also in case anyone is curious, I never seriously considered Haskell as language for `rd`. Haskell makes IO effects painful and tighly controls them and `rr` is one big ball of IO effects. If building a compiler is a dream use case for Haskell, building a debugger in Haskell must surely be the nightmare scenario: IO effects pour our fervently from most functions and methods in `rr` and it all seems unavoidable for the most part. Here OCaml seemed to be a good middle ground and a cool language to potentially write `rd` in. It is a functional programming language but allowed imperative style without a whole lot of fuss also. But, there were other problems with OCaml which took it out of the reckoning, which I will touch upon later.

## Lack of garbage collection was a problem but runtimes are more troublesome

Lets accept it: writing code without garbage collection is a pain. There might be some people who love worrying about allocation and deallocation of memory and the intricacies of various smart pointer types but I'm not necessarily one of them. I want to concentrate on the core logic of the application without having to worry about these issues if I don't have to. Now Golang and OCaml have pretty effective garbage collection and I could have used these languages because as I discussed above, the performance hit in using these languages would not have been too egregious.

But the problem is that garbage collection tends to come with runtimes which makes things extremely inconvenient. Linux signal handling becomes very complicated when you have a runtime. Runtimes also often force layout of data structures be tuned to the needs of the garbage collector rather than the default C struct layout that kernel syscalls like. Now a lot of the time, the rd/rr debugger makes a lot of raw system calls _on behalf_ of the programs it is debugging. So the parameters it is passing to these "remote" system calls need to have specific layout and alignment (e.g. signal handling system calls, socket related system calls etc). Rust allows you to control data structure alignment and packing. You can even ask Rust to give you the same data structure representation as C. But the moment you get into Golang and OCaml, it becomes difficult to deliver the exact memory layout expected by these remote raw system calls. The prospect of using ffi or special libraries everytime I wanted to do something precise/specific made me balk at using OCaml/Golang. 

dlang was somewhat of a special case. It gives you garbage collection _and_ data structure layout is same as C/C++ as I understand it. However, I don't know much about dlang -- it could have been the best of both worlds. In the end I decided to go with Rust because I wanted to experience something really different in terms of the programming-paradigm. dlang to me just seemed like just a better C/C++. 

## Resource management was a good win in Rust

Systems programming involves a lot of acquiring resources and cleaning things up after we're done. In rr/rd, we will often initiate "remote" systems calls in a tracee program which may, say, change the stack pointer prior to making a system call (perhaps to store a nul terminated path). After the lexical scope ends, the remote stack pointer should be restored back to its original value in the tracee/debugee. There are other examples where there is non-trivial resource cleanup at the end of the lexical scope. Golang allows this through the defer keyword (but you could forget to write defer). You can also do this in OCaml by writing a function `withXYZ f` where `f` will get some resource `XYZ` and then `withXYZ` can call `f` and clean up afterwords. But Rust fully embraces RAII style by automatically running `drop` (destructor) at the end of lexical scope which makes porting code from C/C++ to Rust that much easier.

## Lack of Object-Orientation was a concern

OCaml and dlang have powerful OO features. `rr` uses C++ classes extensively. So using OCaml and dlang would have been easy for `rd`. OTOH Golang suffers from an acute lack of abstractions and I balked at writing code to emulate a lot of OO behaviors in `rr` in Golang. Whenever you port a big program to another language it is important to be conservative, in my view. You don't want to make so many structural changes in the way things work in the original codebase while porting things over. You risk things becoming very confusing and the port could fail. In future iterations you could definitely make things more idiomatic in the target language.

As far as Rust is concerned, it does not have traditional OO features. But by using the Deref/DerefMut trait specifically and traits in general I was able to recover and mimic a lot of OO behaviors present in `rr` in `rd`. The `rd` code may not always be beautiful but it works well.

## If it compiles, it usually works in Rust

You may have experienced this: you compile your code successfully in Rust and it will work flawlessly in the first attempt or two. Rust has a reasonably small set of features that work well together. The type inference is top notch and error messages amazingly helpful. Everything is explicit. A data type like i32 does not automatically get coerced to an i64 and unsigned/signed interaction is easy to understand and nothing is implicit. And then there are so many other nice and small design decisions everywhere. All this conspires together to allow fewer bugs to get through in your program. These aspects have been spoken about elsewhere on the internet and I will avoid talking more about them here.

This was a big strike against Golang. If it compiles in Golang, it probably still has a lot of bugs. Golang is quite an underpowered language and it's difficult to encode invariants in the code. A small mistake in the code in `rd` is costly. It could mean that the debugee program could crash millions of instructions later. Its better to invest upfront in preventing bugs by using the type system, generics, pattern matching etc. in advance. OCaml and Rust fit the bill well here. I'm sure dlang would have been good enough too. But I wouldn't have had a lot of confidence in my code if it was written in Golang.

## If you don't have traits/typeclasses in 2020 the language feels outdated

Not having traits/typeclasses was one of the major strikes against OCaml/Golang. Traits/typeclasses allow you to write simple looking code  without getting lost in a sea of different function names and operator symbols. It allows you concentrate on the core algorithm. Enough praise has also been heaped about traits/typeclasses elsewhere so I will stop here. Sufficient to say, I partly chose Rust here because I love traits.

## Debugging the debugger

Given that `rd` is a debugger and I love debuggers in general, it was very important that I could debug `rd` during development. Logging and print statements all have their place but there is no substitute to gdb style debugging in many cases.

I have used the rust debugger (its basically a gdb/lldb backend with customizations for Rust) extensively and I can say it's quite good (especially if you use an IDE frontend like CLion).

Here OCaml was a laggard. Even though OCaml has a debugger it feels quite primitive and has various limitations (e.g. it runs on bytecode only, the OCaml language erases types during runtime so its tough to debug polymorphic functions and so forth). Golang also has a good debugging story but again, the language seems quite unsatisfying to me.

## Social factors played a big role

I wanted to use a language with a large pool of prospective future contributors and Rust certainly fits the bill. Rust programmers are interested in systems/low level programming stuff -- the domain in which `rd` sits. Last, but not the least, some of the prominent contributors to mozilla/rr love Rust too. So it was a no brainer to choose Rust as far as social factors were concerned. 

Golang is also very popular but got eliminated due to other factors discussed above. This leaves OCaml and dlang. OCaml is niche language and I have a soft spot for it. It has blazing fast compile times (Rust are you listening? :-) ) and in general it is a well-tended language. I believe OCaml has a very bright future ahead of it (even though few people use it today). dlang for some reason just never made a strong impression on me and I didn't get inspired by any dlang projects on Github -- maybe I should have looked harder/longer.

# Conclusion

This summarizes all the pros and cons that went into choosing Rust as the target language for the mozilla/rr port. I'm happy with the choice I made!
