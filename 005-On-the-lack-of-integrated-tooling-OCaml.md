_11 August 2021_

### [Other writings](README.md)

#  Thoughts on Ocaml & Haskell and OCaml's (supposedly) pathetic state of tooling

## Prelude

I'm primarily a Rust programmer but I like dabbling in Haskell and OCaml occationally. I'm attracted to OCaml's simplicity and practicality as much as I am attracted to Haskell's sheer brilliance and take no prisoners approach in wanting to do things "correctly."  

Now, for a long time I used to think that all data mutation in programming was evil but over the years my views have mellowed a bit. My current opinion is that avoiding data mutation leaves a lot of performance on the table. Also, if you dissallow mutation a lot of simple algorithms become complex. Mutation done right can be really helpful. In fact, all the current CPUs are well optimized for mutation. So if you're using really "pure" Haskell and avoiding mutation everywhere you are (a) Often making things more complex than they need to be (b) Missing out on a lot extra performance. In fact using Rust has taught me to love data mutation again. But the problem with Rust is that it does not have GC. If you need the convenience of a GC and don't hate data mutation, OCaml can be a great choice.

As my views on mutation have mellowed, so has my opinion on OCaml gone up. Yes, OCaml does not have higher kinded types. Yes, OCaml does not have many useful functional language features of Haskell especially typeclasses. But OCaml does allow mutation. And it's done in a tasteful way enough. Haskell fans may chime in now and say that mutation _is_ possible in Haskell too. Yes it is but it's simpler in OCaml. And because OCaml does not go out of its way to avoid mutation it does not need many of the complex functional language patterns that had to be invented for a language like Haskell. Yes, a lot of innovation that occurred in the Haskell space but when you just need to make a simple piece of software maybe you shouldn't have to worry about Monad transformers and other more complicated things. My view on Haskell currently is that it still is a brilliant language but it's not the "promised land" that I once thought it to be. In fact, as I've matured as a human, I don't think there are any promised lands at all. Use different languages for different situations and try to appreciate all of them -- you don't need to be part of one camp or the other. But I digress...

This post is about mostly OCaml tooling and I'll get to that soon enough. But I want to summarize my feelings towards OCaml: I feel that OCaml deserves more love from the programmer community that it currently gets. If you need extreme performance then use Rust. If you really want to do pure functional programming then use Haskell. But if you want a practical GC-ed functional language with a blazingly fast compiler then use OCaml. Did I forget to mention the insanely fast compiler :-) ? 

And finally yes, the elephant in the room for those who follow OCaml from a distance: OCaml multicore is really on the horizon. It's inching closer every month. It will happen. For those who don't know, OCaml has a lock in its runtime system a bit like Python does. This lock does not allow threads to execute OCaml code parallelly in the OCaml runtime (it allows them to execute concurrently but not parallely). In practice this is not a very big deal because while you have a OCaml thread blocked on a system call another OCaml thread can be executing. What happens currently is that before a Ocaml thread makes a blocking system call it can release the runtime system lock allowing another OCaml threads of execution to run. Another way of saying this is that if you're IO bound, the OCaml runtime should not be your bottleneck. If you need to do parallel math calculations and similar things then you'll definitely need OCaml code to be executing parallely on many cores (hence the eagerly awaited OCaml multicore project). The state of the multicore project is healthy. You can install it [locally](https://github.com/ocaml-multicore/multicore-opam) and play around with it _today_ and it is intended to be fully backwards compatible with current OCaml code. (P.S. you want to use the `4.12+domains` version). The next major version of OCaml (OCaml 5.0) will have multicore builtin. When will it land? I don't know enought to make a guess but there is a good amount of activity. I would suggest checking the monthly multicore project updates on https://discuss.ocaml.org to get more information.

## Ok OCaml is good -- but isn't OCaml tooling horrible?

There is a current trope that while OCaml the language is decent the tooling is horrible, attrocious and incomprehensible. The problem is that we've been spoiled by new generation of languages like Rust that have learn from their predecessors and built some great integrated tooling from day 1. `cargo` is a best of breed language specific build tool. It handles package management, building, documentation. All these things are handled by separate tools in some of other older languages.

Such is the case with OCaml. In its long evolution from 1996, it has had many build systems and ancillary tools. The growth of tooling has been organic and not resulted in a monolithic, super consistent tool like `cargo`. Its unfortunate but this is a legacy issue. But in 2021 I will recommend:

### The OCaml Tooling Cannon for 2021
- Dune as your build tool
- Opam as your package manager
- [`ocaml-lsp-server`](https://github.com/ocaml/ocaml-lsp) as your LSP server (this has an associated vscode plugin if you're into VSCode), You can just use the lsp server directly if you are using Neovim (after you make the appropriate configuration in `lsp-config`). If you use Kakoune, then install the `kak-lsp` plugin which allows you to use a variety of LSPs with Kakoune (including the `ocaml-lsp-server`). 
- `odig` & `odoc` to build documentation locally. (`odig` just makes things simpler; just issue `odig odoc` on the command line)
- [docs.ocaml.pro](https://docs.ocaml.pro) as your online documentation tool (like [hoogle](https://hoogle.haskell.org/) for Haskell). This is a relatively new addition and fills a big gap in the community. Ideally it should get official blessing and move to ocaml.org as maybe oogle.ocaml.org :-) (with prominent credits to OCamPro that took the intiative to get this done).
- `utop` as the Haskell `ghci` equivalent (not as versatile as ghci but quite nifty for the limited things that it does do)
- `ocamlformat` for formatting source code. It's best to setup formatting on editor save. This really speeds up development for beginners especially 
- A Unix like operating system. Basically, avoid using (pure) Windows for OCaml development. It's likely to be a painful experience if you are a beginner. Here I would recommend installing a Linux VM in Windows, using something like WSL or dual boot. (MacOS is also great for OCaml in case you're wondering). The pain of using OCaml in Windows in my view is really reflective of the overall decline of Windows as a development platform because I notice the same story is repeated across other languages and tools -- I've seen so many tickets across GitHub from Windows users in many different projects. Its unfortunate but this is the reality. The reason is simple: since the cloud is basically Linux based it makes sense to develop on a Unix like platform. Even on the client side (Android and iOS) you basically have Linux/Unix. So to me at least it does not make any sense to do development on Windows anymore (unless you're doing something Windows specific).

Now, it was important for me to put the above list of recommendations out there because there are a lot of resources on the internet written at various points of time that say various things. I'm putting this list out there so that people who are starting out in OCaml can achieve nirvana without getting caught up with obsolete religions like ocamlbuild, raw merlin etc. ;-). 

If you follow this list then hopefully you won't curse OCaml tooling very much because each of these tools I've mentioned are quite mature, well documented and mainstream in the community. 

Of course, the fact that there are different tools in OCaml which accomplish what a single tool like `cargo` (and its plugins) does can be frustrating at times. If you're able to persist beyond the initial learning curve I promise you that things are not so bad at all. In fact there are some cool things that you can do in dune and opam. As an example, dune is a composible build system. If you include the source code of another dune built package in your source code you will automatically be able to build (and develop) both packages simultaneously. This is called vendoring and it really simple to do in dune. Opam allows you to use many package repositories really easily and "overlay" the repositories over each other a bit like `nix` does. The support for various compilers being installed simultaneously is also very good and intuitive. I'm not saying that these features don't exist in other languages -- its just that opam/dune do many things quite well. So what if there is no monolithic tool like `cargo`?

### Other questions

- Should I use the ReasonML syntax? If you're doing backend development then stick to the traditional OCaml syntax. You will have a bigger community to work with. In my view, surface syntax, while important is really a small aspect of the overall part of programming in an ML like language. OCaml syntax while a bit weird is not so bad. In fact, OCaml syntax is closer to Haskell syntax. So if you have a desire to add Haskell to your mix stick with OCaml syntax
- Should I use things like esy instead of opam? No. Use opam if you're doing backend development

## OCaml needs more pioneers

So my position on OCaml is this: it would be an ideal situation if OCaml had unified tooling. But due to legacy reasons it does not and barring some suprising developments we are likely to have a menagerie of tools in the years to come. Is this the reason why OCaml is not more popular? Does this tooling situation doom OCaml?

As a point of comparison, for many years Haskell also had a really bad tooling story also. Build times were atrocious (and continue to be atrocious). There was dependency hell (lovingly called "cabal hell" named after the Haskell specific build tool `cabal`). IDE integration was bad. I would posit that the build tooling situation was even worse than OCaml: because OCaml compilation is very fast if you ever encounter a dependency problem you can always do a build from scrach build again. But in Haskell's case you may have now lost ~15-30 mins of your time. 

As a point of comparison Haskell also does not have a unified tool ala `cargo`. It has stack, it has cabal, it has other interesting things like hpack which produce cabal files. In the backend stack still uses cabal AFAIK. Yes everyone does work quite well by now and stack and cabal both allow sandbox builds avoiding "cabal hell". Stack also allows sharing build artifacts across projects -- it will use this cache to save time compiling from scratch. But you can always encounter strange inconsistencies and you have to repeat yourself when building things in Haskell. Haskell also has a really nice (but separate) documentation tool called haddock (unlike `cargo` which again, you guessed it does documentation too). As far as the IDE support is concerned only recently did the Haskell community coalesce around a IDE solution that is capable and performant. There were many failed/short lived IDE experiments along the way to the present Haskell Language Server.

The point here is that Haskell tooling has been erratic and poor in places for years but yet the language seems to attract much more interest and talent than OCaml based on my unscientific assessment. The answer is that programmers went in and made some cool software and didn'nt bother about how many other people were using Haskell. These cool projects attracted more programmers. So a language that had (a) A bad tooling story for many years (b) Was actually quite tough when compared to OCaml became more popular than OCaml.

The reason for OCaml's niche status that for some reason or the other it has not be able to attract the non-institutional programmer. By non-institutional programmer I mean outside of academia (INRIA, Cambridge, French universities) or select industry circles. This is beginning to change as people search for better alternatives than the current crop of mainstream languages. For OCaml to achieve more "success" (at least as GitHub defines it -- more programmers, more stars, more repositories, more contributions from people not directly associated with a project) the current crop of OCaml programmers simply have to build some cool projects. It's tough, but if there are some breakthrough projects, OCaml will attract more programmers thus leading to a positive loop that Haskell has already experienced.

This is not to say that OCaml does not already have some cool projects -- it does. But for some reason or the other they have not caught the popular imagination. Once that happens, programmers will be more interested in being part of some of these cool projects and work past the fragemented tooling story. In any case, the tools are reasonably mature and get the job done in 2021 in a way they didn't in 2016 or 2011.

### OCaml is brilliant too

I called Haskell brilliant. And it certainly is. But people don't realize that OCaml is brilliant too: it's a well balanced sytem. That means: it produces decent sized binaries, does not need wizard level tuning to get good performance and provides a fast feedback loop during development. What use is the brilliant type system of Haskell if a decent sized project needs half hour to compile fully? What use is it if produces binaries can be 100M+ regularly? What use is it if you can suddenly get mysterious space leaks and need expert level debugging to find why and where they are occurring? This imbalance of Haskell is something that I think needs more attention from the broader community when making an assessment of the language as whole.

### But libraries ???

OCaml has decent set of available libraries. Yes, its not as good as say Golang (or even Haskell) but there are plenty of packages available. Again it boils down to the title of this section "OCaml needs pioneers." There was a time when Haskell also did not have many libraries but people came to the language attracted by what they thought it could do for them without worrying about how many stars their repos would get and **built that software**.

So, hopefully exited by the interesting set of tradeoffs that OCaml has made, I invite you to learn OCaml, not worry about how many stars your repo in OCaml will get (it won't get too many right now) and simply for the love of the language build something cool!
