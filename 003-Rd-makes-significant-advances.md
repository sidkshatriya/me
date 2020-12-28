_28 Dec 2020_

### [Other writings](https://github.com/sidkshatriya/me/blob/master/README.md)

#  45,000+ lines of Rust code later: An update on the Record & Debug Tool (`rd`)

The Record & Debug Tool ([`rd`](https://github.com/sidkshatriya/rd)) is a record/replay debugger written in Rust.

Its been been about 4 months since my last writeup on `rd`. `rd` is an effort to port the fantastic [rr](https://github.com/rr-debugger/rr) project from C/C++ to Rust. Why is this a good idea? Why Rust? For that, please read some of my older posts:

- [12 Aug 2020 - Introducing rd: A port of mozilla/rr to the Rust programming language](https://github.com/sidkshatriya/me/blob/master/001-rd-intro.md)
- [13 Aug 2020 - rd: Why I chose Rust instead of Golang, OCaml or Dlang for the mozilla/rr debugger port](https://github.com/sidkshatriya/me/blob/master/002-why-rust.md)

Today, I'm going to give you an update on the progress that has been made since the above writeups in August 2020. I will also talk about how the porting effort is progressing, my thoughts on Rust, some tips and suggestions if you're about embark on your own project, and more.

For those people who would like a quick update on what is working and what is not, please see [`What works`](https://github.com/sidkshatriya/rd#what-works).

## Update

The biggest change since August 2020 is that `rd` can record program executions just like `rr`. A viable way of using the project now is to record your program execution with `rd` and then replay using with `rr`. (`rd` supports replay of program traces but in non-interactive replay mode only, currently). 

What does "interactive replay" mean anyways? Interactive replay means that you can use `gdb` while replaying your program. In traditional debugging you are only able to run the program forward. `rr` allows you run the program forwards and backwards while doing the usual things like setting breakpoints, inspecting the stack, looking at variables etc in the gdb frontend. `rr` does this by presenting a backend engine to the `gdb` front-end. This work is still incomplete in the `rd` port. However, `rd` has  non-interative replay (`replay -a`) which causes the whole register accurate replay to actually happen internally.

The heart of the `rr` project is the record & replay logic which has successfully been ported over. The `gdb` backend and related functionality is a relatively smaller piece of code (compared to say the record/replay logic). In other words, the completion of this porting project is in sight!!

Now that the update of the project is out of the way, I'd like to share some of my thoughts, feelings and motivations after having worked on this project thus far. I will also share some tips and suggestions in case you're thinking of embarking on a personal project.

## Port a project to understand it

One of the reasons why I embarked on this long (and somewhat lonely) journey was actually to increase my knowledge of how the record/replay debugging system was implemented in `rr`. I was so in love with `rr` that I wanted to understand it on a deeper level. What better way to understand it than translate the code base over to another language?

The traditional path to mastery and knowledge is to implement a substantial feature in the `rr` project. But the thought of wrestling with the mess that is C++ and the segfaults that come with it was not appealing to me. Rust seemed like a wonderful choice and I've spoken about that in my older posts.

## Stop looking at the goal to make progress

The `rr` project contains the toughest code I've ever read. Its intimidating. Its a huge project with tens of thousands of lines of C/C++ code written by some absolutely brilliant engineers. There were many moments that I just thought I should give up. But I kept persevering and now I can see the finish line ahead. There is a good amount of work still left on this port but I think the toughest challenges have probably been surmounted. 

The trick that I used throughtout this project was to stop thinking about the end goal. I avoided thinking about where I am on the project. I just kept my head down and kept working on it. The key learning here is that you sometimes need to fool yourself and not think about how much of the journey is left. Because if you do that, then you start panicking and wondering if its going to be possible.

Today `rd` has 45,000+ lines of ported over Rust code! This is the power of incremental progress

## Try to keep your effort consistent rather than "peaky"

Another approach which I followed on this project was to avoid a peaky work schedule. In some of my older personal projects I would get carried away and work like crazy only to crash and get exhausted. It would take me a day or two to get back to the project. In the end, the average productivity would not be so high. In this porting project, I tried to emphasize incremental progress. Yes, there were a lot of days in which I was inspired and kept going but I'd always try to end at a point where I knew I had enough emotional energy to work the very next day on the project.

## Don't let perfect be the enemy of the good

A port is a messy venture. Especially from C/C++ to Rust. The `rr` codebase is Object Oriented while Rust is not. So its been quite a task to translate that to Rust. Also Rust places so much emphasis on borrowing and ownership that the code sometimes feels polluted by those considerations. Due to these two factors I often felt that the Rust codebase was not as succinct as the C/C++ codebase. I sometimes feel that I needed to produce a more elegant Rust codebase. But in my experience its better to get things working and then progressively improve things. My ongoing goal is to continue making the port more idiomatic Rust and improve code elegance but my emphasis is first on getting things working. Perfection can wait.

## If it compiles, it works

One of the superpowers of Rust is that if it compiles, it will usually work. This has been invaluable to me because Systems Programming is extremely unforgiving. A small mistake and your program will not work. Because Rust is so explicit and its type system so amazing I continue to be suprised at how easily I was able to avoid bugs or stamp them out if they cropped up.

## Test suites are valuable and so are logs

Another one of the reasons why the port has been able to make so much progress and is generally quite robust is the original project, `rr`'s amazing test suite. Usually, C/C++ codebases don't have too many tests. `rr` bucks this trend and it has 2000+ tests that exercise its record and replay system. I used this test suite with `rd`. Pretty soon, it becomes a game -- every additional test which `rd` succeeded on became a brain reward for me and an opportunity to work on the next bug or feature (or "puzzles" as my brain started viewing them).

So if you're looking to port over a project to <your-favorite-language> make sure there is a large test-suite. Not only will it help you with robustness and correctness, it will become a fun "arcade" game as you keep working on boosting your test pass-percentage.

For completeness, I want to also appreciate `rr`'s amazing logs that let you know what the program is doing as its records/replays execution of a program. These logs were invaluable in solving bugs in the `rd` port. I would simply compare the logs produced by `rd` and `rr` in many situations and this has helped me solve some complex bugs in the `rd` port.

## A "Shout Out" to the kakoune modal editor

As the `rd` codebase grew larger and larger it became impractical to continue using some of the traditional editors/IDEs I started the project with. Things became simply too slow.

The main combination I have been using for some time now is the `kakoune` editor with Rust Analyzer (via the `kak-lsp` kakoune plugin). 

`kakoune` is a modal editor like `vim/neovim` but better (in my opinion) in a few fundamental respects. 

In `kak` you make the selection first and then select the action (rather than the other way around as in vim). This simple but revolutionary difference makes modal editing so much simpler (and I would even say more powerful) than vim. Please checkout [kakoune.org](https://kakoune.org). The kakoune editor is a gem. As software developers, we're all quite passionate about the tools we use and I'm very passionate about kak.

Like all modal editors, mastery will lead to a lot of time saved. Also because the editor is running in a terminal, things tends to be faster compared to browser/java based IDEs/editors. In Rust, things are already quite slow when it comes to editor completion/code checks so using a terminal based editor can give you that extra boost of responsiveness.

I will write a separate post one day singing paeans to the kakoune editor :-) ! In the meanwhile here is a great [article](https://kakoune.org/why-kakoune/why-kakoune.html) on Kakoune by its creator.

## What is preventing Rust from achieving total world domination?

I've loved Rust more and more as this port has progressed. In my opinion the only thing really holding it back is that the IDE/compilation experience is still too slow for my taste. Rust Analyzer does a good job and I'm really happy with it. However, I'd like it to be faster. I'd like Rust compilation to be a bit faster too. (There are some workarounds like breaking your project into multiple crates etc. I haven't heeded that advice yet) 

I'm constantly reminded of the speed difference when I look at the equivalent `rr` C/C++ codebase. Everything is so lightning fast from the IDE/editor experience to compilation in comparison to Rust! Hopefully Rust will get there. During the course of the execution of the port, things have progressively gotten better in this area but there is still a long way to go...

I hope you liked this update. Please check the [project](https://github.com/sidkshatriya/rd) out and play around with it! 

### Comments
You can add your comments on this [reddit post](https://www.reddit.com/r/rust/comments/klnddg/45000_lines_of_rust_code_later_an_update_on_the/)

_P.S. I'm currently looking for opportunities in the Rust and/or Systems Programming space. I'd love to speak with you further if you know of any. Please check my GitHub profile for contact details._
