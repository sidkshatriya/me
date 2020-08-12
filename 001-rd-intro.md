_12 August 2020_ 
### [Other writings](https://github.com/sidkshatriya/me/blob/master/README.md)

# `rd`: A port of mozilla/rr to Rust programming language

[mozilla/rr](https://github.com/mozilla/rr) is an excellent record/replay debugger written in C/C++.

Over the last few months I've been working on porting rr to the Rust programming language. The port is still in-progress but many things work already. 

The project is called "The Record and Debug tool" or "rd" for short. The [repo](https://github.com/sidkshatriya/rd) is available on GitHub. It already has 30k+ lines of ported over Rust code.

I was somewhat inspired by the technical discussion in a mozilla/rr [ticket](https://github.com/mozilla/rr/issues/2181) about porting rr to Rust even though at that point in time, I was not really a fan of the Rust programming language (though I am now!)

I hope to write a longer post about my experiences while porting rr to Rust. For today I just want to share some of the reasons why I embarked on this journey.

## Background
I've always found record/replay systems very powerful and amazing. It's so frustrating to have to restart the gdb debugger again and again. Also it's not always possible to reproduce a bug again. 

In record/replay debugging you record the offending execution only once and then you can replay (and debug) the same _exact_ execution again and again. This is extremely tricky to accomplish in practice because many system calls are not deterministic. If you try to run any program again, then you might get a different random number, get memory allocations at different addresses, get different results to various system calls and so forth. Record/replay systems record all sources of non-determinism during the record phase of a program and feed the same results during subsequent executions or "replay" to the program being debugged.

There are many record/replay systems available. But many of them either support a single language (e.g. Java) or cost a lot of money. `rr` is a well maintained and robust record/replay debugger. What is amazing about it is that it's performant and free. It probably gives many proprietary systems a run for their money too.

I've been mesmerized by rr for many years now. Though I've had some minor contributions to rr, I was always been repelled by its C/C++ codebase. I thought that it might be amazing if rr were re-implemented in a modern, clean and performant language. I flirted with the idea of implementing rr in golang, ocaml or even dlang. But in the end I chose Rust as opposed to the other mentioned languages (again that could be the subject of another post).

In this post I want to explain why you should be interested in a Rust port of a program (`rr`) that already exists and works well.

## Why port rr to Rust?
If you're interested in rr, here are the main reasons that capture why it is a good idea to port rr to Rust and why you should be interested in this project:

### Reduce complexity, increase reliability
C/C++ is a complex beast. As the Linux kernel gets more and more complex and gathers even more quirks, the rr codebase gets more and more complex too. With Rust the hope is that we have a clean and modern language that allows us to manage the complexity of record/replay. Of course, we still need to deal with the inherent complexity of record/replay but Rust helps with writing reliable code and doing refactorings with more confidence. 

### Reduce barriers to understanding and contribution
Once you understand the core principles, Rust can be easier than C/C++ to grok. Hopefully this will allow more people to inspect, improve and offer fixes to the rd codebase.

### Provide an alternative, compatible implementation
Just like there can be multiple compilers for the same language, it might be a good idea to have multiple compatible implementations of rr. This might help with debugging weird bugs and optimize various implementations around different parameters.

### Be a playground for experimental features
This is something for the future. The hope is rd can become a playground for experimentation and implement some innovative features in record/replay. Also rd has access to the awesome Rust cargo ecosystem which means that functionality already implemented elsewhere can be added to it much more easily.

### Be more memory safe
As rd is written in Rust, the hope is that the implementation is more memory safe. However, as rr is mostly run in environments where it's not critical to have memory safety so this may not be valuable to most users. However, it's possible there might be usage scenarios where the memory safety of Rust might be valuable.

## What's next?
Please [check](https://github.com/sidkshatriya/rd) the project out and play around with it. Let me know what you think. Contributions to the `rd` project are welcome!
