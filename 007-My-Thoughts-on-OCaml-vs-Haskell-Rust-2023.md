_26 April 2023_

### [Other writings](README.md)

#  My Thoughts on OCaml vs Haskell/Rust in 2023

Recently, osa1's [My Thoughts on OCaml](https://osa1.net/posts/2023-04-24-ocaml-thoughts.html) generated quite
a [robust](https://news.ycombinator.com/item?id=35699697) conversation on Hacker News. Overall I felt the blog post was
a bit too critical about OCaml. However, everyone has a right to their opinions and I respect whatever has been
written.

Except for a couple of points, the article _didn't_ resonate with me, so I
thought I should pen down my good/bad experiences with OCaml and see if others have felt the same way as me.

I've used OCaml for some time now and have tried it in some internal projects of mine. The experience
has always been net-positive. I've been wanting to document my experiences but somehow never found the time.
osa1's article on OCaml propelled me to author this blogpost more urgently.

My current area of work involves a lot of linux "systems programming". While it is possible to do such work
in OCaml, it is not ideal. Here Rust simply excels and wins hands down, so I use the Rust language instead.
Systems programming goes really well with linear types. Now Rust provides affine types (which can provide many benefits of
a linear-type-like system). OCaml neither has linear nor affine types, sadly. Even though Haskell provides unboxed types,
nice primitive integer types, proper linear types (so on paper is nicer than OCaml in many respects for systems programming)
the evaluation model and type system around IO is just a bit too impractical/tedious for me.

For systems programming I would say Rust > OCaml > Haskell. It is no surprise I use Rust.

In this blog post I would like to document my experiences with OCaml as of 2023 and provide some comparison points
with Haskell. I will also talk about Rust where I can.

To me, the lack of a GC is the main differentiator between Haskell/OCaml and Rust. 
In general, I would say that if you _don't_ need the convenience of garbage collection then
just skip OCaml and Haskell and just use Rust. You might want the convenience of a GC when your application _does not_ 
have very high performance requirements and/or when the shape of your data structures in memory can be complex. Examples of complex data
structures would be self-referential data structures, data structures in which there are bidirectional pointer
links, recursive data structures etc. Another area where I would strongly consider using OCaml is in the async 
space. I love Rust but I absolutely abhor the complexity it has spawned in the async space. Async Rust is 
truly a beast and I don't mean this positively. I will try to avoid Rust in those situations. Rust with vanilla system
threads is extremely pleasant and wonderful. However, async Rust (Tokio and friends) is still a no-no for me. Another area where you might
_not_ want to use Rust is if your program provides some sort of simple interpreted language or DSL. With OCaml (and Haskell)
that DSL/interpreted language would get OCaml/Haskell's Garbage collection "for free." Now any sufficiently complex interpreted
language has the tendency of creating cycles in data structures. And if you wrote any interpreter in Rust you would need to
worry about collecting the garbage while the interpreter ran and remove any cycles. Rust uses reference counting a lot
and cycles are the enemy of reference counting. In short, OCaml/Haskell lend themselves very well to creating 
interpreters, and you may then want to avoid Rust. You could write a GC in Rust itself but that is another level of
complexity.

## Why do I want to compare OCaml with Haskell?

Simply because OCaml and Haskell languages are similar in many ways. 
Haskell is a very popular language in the functional space, it is packed with features, has an active and prolific
community with an endless series of blogs and
cool libraries. It is natural why Scala, F#, Kotlin, OCaml, (insert functional/functional-ish language here) would
like to compare themselves with Haskell.

Before I get deep into the guts of this blogpost -- for those who are wondering -- what about Scala? Or F#? 
While I find Scala quite featureful, it is built on the JVM.
Because of this there are some compromises that have entered the design of Scala: Generally speaking, `Null` can appear
where an `Object` can and this is deal killer for me. `Null` issues are more of a problem with Java packages I understand,
but given that you can end up using lot of popular Java packages with Scala this is still a concern. Scala's frankenstein
object-functional nature is also too complex for my personal taste. People love F# too but the problem of `Null` exists on
the CLR too. `Null`/`Nil` are truly "billion-dollar" [mistakes](https://en.wikipedia.org/wiki/Tony_Hoare). In that respect OCaml/Rust are ideal (Haskell has issues
with `undefined` which does not make it totally "good" in this department). 

Finally, there are just too many layers between me and the machine on the JVM/CLR for my personal taste. 
Haskell/OCaml/Rust feel more grokkable to me.

I've divided the rest of the article in more digestible headings in which I pick a topic and talk about strengths/weaknesses
of OCaml and Haskell (and sometimes Rust) as of 2023.

## The lack of ad-hoc polymorphism in OCaml

Here I generally agree with what osa1 has written in his blogpost I've linked above.

Lack of ad-hoc polymorphism is truly OCaml's biggest weakness. For someone who likes and sometimes even feels "love" for OCaml
there is no other way to put it. It is a pity that Haskell's class/instance and Rust trait/implements have
no real counterparts in OCaml. When you have a data structure in any language, you would like to know
what "operations" that data structure supports. Yes, OCaml does have functors, modules and module signatures. They can 
achieve the same outcome but suffer from one major problem: they have an open world assumption. In Haskell/Rust
a data structure *says* that it implements a particular trait/class. It is opt-in. But in OCaml any module that satisfies a
signature required by a functor will suffice. This hurts documentation. In Rust I can see cargo
documentation for a library that I've not written, and at a glance see what traits a data structure implements. 
Same with Haskell. In OCaml I have to `grep` for things like `.Make` to see
what functors have been instantiated or look at massive module signatures to see what operations are supported.
(Note that `.Make` is only a convention so this whole process is unsatisfying). A good (but imperfect) 
analogy of what we have in OCaml is "duck typing" that Golang also has. In Golang any data structure that supports the same
methods as an interface satisfies that interface. But then it would be useful to know which data structures
satisfy which interfaces in a more direct way!

### Ad-hoc polymorphism promotes interoperability, establishes common standards/common vocabulary

One thing that I'm struck by is how ad-hoc polymorphism promotes code readability and common standards across the
language ecosystem. There are some famous Haskell type classes like Monad, Monoid, Functor, Applicative etc. In Rust
we have type classes for iterators, errors, type conversions, data structure pretty printing
etc. These type classes establish a common "vocabulary". Once I see a Monad in Haskell, or a `AsRef<T>`, or a `From<T>`
etc. in Rust I instantly know exactly what operations I have available to me and generally what to expect. 
In OCaml I have to rely on fragile code/function naming conventions or peer deeply at the module signature/module implementation.
It is just not as satisfying.

Rust and Haskell library code, because of the common vocabulary established by popularly used traits/type classes
can look quite uniform. OTOH each OCaml library's code structure can be quite custom. You actually have to
read the code more thoroughly to get a feel of everything. In Rust/Haskell I can often just look at the
cargo/haddock documentation. OCaml does have Odoc which helps a lot but the overall shape of the OCaml 
language does not lend itself to learning too many high level things about a library by exploiting this "common vocabulary"
that ad-hoc polymorphism promotes. Note that OCaml *does* achieve "common vocabulary" in various other ways but it is just not
as rich as that of Haskell/Rust.

### Is this lack of ad-hoc polymorphism fatal to OCaml?

I think not. If was so, I would not use OCaml. OCaml has so many other strengths and I hope to do justice to
many of them. This is surely going to be a tall order because I've been quite critical of OCaml so far.

# The sheer practicality of OCaml

I own and have read a lot of Haskell books. I've also loved reading some Haskell papers and
many blog posts over the years. My undergrad degree is in Computer Science and I actually love hard-core PL/Computer Science-*y* stuff. Haskell has been a major influence on me.

I could *read* a piece of Haskell and my heart dances with joy. These moments of joy are not as common with OCaml. 

But then I just feel a bit exhausted with Haskell. Maybe this is just me. Writing Haskell
with Monad transformers, worrying about the complexity of Haskell exceptions, errors, laziness etc. all seems overwhelming. 
Whenever I wanted to prototype something, build something I do think of Haskell as a language to try out. But in the end I always end up
choosing Rust/OCaml/Python. I think Haskell is just too impractical for my domain. For the kind of programming I do (Systems programming)
everything involves the IO Monad. But even if I were not doing system programming, I find laziness in Haskell quite 
debilitating. I totally buy and appreciate how laziness increases language expressiveness. But I just don't have the time
or inclination to look at memory usage graphs and squash laziness related memory leaks that can be quite subtle. Removing
laziness is not as trivial as slapping on a `!` or adding a GHC language extension. Here is an interesting blog post
[Strict vs Lazy](https://www.tweag.io/blog/2022-05-12-strict-vs-lazy/) that goes deeper into the issue. 
[Make invalid laziness unresentable](http://h2.jaguarpaw.co.uk/posts/make-invalid-laziness-unrepresentable) is another gem.
Sometimes to prevent laziness from becoming a problem you must use a (nevertheless beautifully written) library like [foldl](https://github.com/Gabriella439/foldl).
Still feels excessive.

I urge you to look at the tickets on big mainstream Haskell projects like the Haskell language server, the GHC compiler itself.
You will find many tickets in which space leaks arose due to some innocuous looking code and Haskell experts needed to resolve them.
I am definitely not that expert (and do I even want to be?). In general as a human it will always be difficult for me to recognize
all patterns and situations where laziness creates a problem in advance. The debugging will have to happen during runtime 
and I don't want to do that.

In general, I have been frustrated with the strange dichotomy of the inordinate amount of time Haskellers spend
encoding invariants by using the excellent Haskell type system so that they don't get runtime errors. But then they
strangely tolerate the possibility of space leaks that might occur during the same runtime. I initiated a discussion
on reddit on [How can Haskell Programmers tolerate space leaks](https://www.reddit.com/r/haskell/comments/pvosen/how_can_haskell_programmers_tolerate_space_leaks/)
and in true Haskell community style got a lot (positive) participation and interesting/useful comments.

Just like I feel that the lack of ad-hoc polymorphism is OCaml's achilles heel. Laziness could be Haskell's. Laziness
no-doubt helps you *write* beautiful programs. But laziness certainly doesn't help you *run* beautiful programs. One
common feedback in the complaints I've made is that laziness is not a problem in *practice*. I must confess I've not written
enough big, long-running Haskell programs to disagree with this statement.

I must point out here that Purescript is an excellent example of a Haskell-like language. It has many of the syntactic benefits 
of Haskell but it is strict by default. With laziness gone, Purescript cannot possibly be as beautiful as Haskell but this could represent
a compromise. Anyway, right now Purescript is super-super niche, compiles to js (unappetizing!) so it has never really been a viable option for me.

# The anemic nature of the OCaml standard library

This criticism of OCaml seems to be a meme. People complaining about the OCaml seem to talk endlessly about the OCaml standard library. In general,
on a day-to-day level the inconsistencies and "infelicities" (in Haskell verbiage) of the OCaml Standard library don't
affect me that much. Maybe I'm not a compiler engineer working on something deep but the inconsistencies in the OCaml
standard library never really have affected me. Sorry for sounding like a simpleton but that is just the way I feel.
What people have forgotten is that the pace of development in OCaml has really speeded up. Some papercuts have been removed
in recent years. The cool thing about this is that this is not a big problem like lack of ad-hoc polymorphism. The
standard library can be improved, imperfect methods can be deprecated and new ones added. I wouldn't worry about this if I'm picking up OCaml today.

# Funny parsing issues / syntax

Yes, OCaml does have a funny syntax. You have `int list` rather and `list int` and so forth. In that respect OCaml is 
similar to C/C++. In C/C++ we have `int*` rather than `*int` (which would be more like Golang, Rust, Haskell). 
Once you get used to it, this falls into the background. The surface syntax is just such a small aspect of the large part
of the art of the programming. Then we have inconsistencies in the way major control structures are written in OCaml. All
these feel small to me and have faded into the background ever since I spent just a few hours with OCaml. I just keep
an OCaml syntax sheet near me if I'm coming back to OCaml after a few months.

People seem to complain about parsing issues/ambiguity issues in the OCaml language also. Sorry for sounding flippant but 
in the day-to-day usage I've never really encountered a real issue that could not be fixed by a bracket. Again, note 
that this is not as problematic as a Haskell program that could leaking memory during runtime because you forgot to 
take into account something subtle. 

# The debuggability of Haskell vs OCaml

One thing that I've not really seen mentioned very much is that the Haskell is fundamentally undebuggable. Here, by "debuggable" I mean
debuggable by a classic debugger in the style of a `gdb`. Now the Haskell repl has some breakpoint setting abilities.
Since Haskell is a lazy language, the very act of observing a variable value 
in a Haskell debugger means that you might evaluate a thunk that would ordinarily not be
evaluated at that point of the program. Eagerly evaluated languages like Rust, C/C++, OCaml etc. don't suffer from this problem.
So if you want to see what is in a variable you will disturb the program. (A bit like Quantum Mechanics but I digress :-) ). 
Now, I love debuggers and to me this is another
major issue for me with Haskell. You *could* do sophisticated things like evaluate the Haskell variable in rr "diversion session" that will
essentially (unix) fork the program and not disturb the main program (BTW this would be an excellent project to do for Haskell). 
But then if evaluation requires IO (and it could) the rr approach again
becomes problematic. Anyway, debugging via gdb style debugging is really not the way one approaches bugs in Haskell but
the laziness does get in the way of this, if we every wanted a proper `gdb` style debugger for Haskell.

What about OCaml? Debugging is area is where OCaml could have capitalized on but sadly does not.
OCaml has a byte code debugger called `ocamldebug` and it is sadly one of the worst debuggers in terms of usability I have encountered.
It's a pity that the OCaml community has not invested enough time and effort into
the debugger. Because the language is strict, there is no reason why OCaml cannot deliver a good debugger. 

As mentioned, `ocamldebug` works on bytecode (which is not a problem really BTW -- you could just temporarily switch to bytecode
to debug your problem). But there was also an effort a few years ago to integrate OCaml into gdb (like Rust has). 
That project was abandoned for various reasons. OCaml could have provided excellent integration with gdb if the community
put some effort into this.

Here is one issue that I've seen in OCaml -- because it is a small
community we often see the carcases of many a promising project. Either the programmers moved onto other things or the
Ph.D./Project was over and that promising line of work was abandoned. In Haskell I also see that but there is a higher rate of "completion" of features. No doubt this is unscientific assessement based of "feel".
Haskellers seem to want to put a nail on things more than OCaml. This could just be because there are more people in the Haskell community or could
reflect an attitude in the community of getting things done "fully" and "correctly". I don't know -- this is something the OCaml 
could learn from Haskell more -- completing things. In Rust, simply because the language has become so big/mainstream that everything already 
seems so polished and complete.

But again it is not all rosy in Haskell land. I also want talk about something that certainly shocked me: **it is not possible to get an accurate backtrace in a 
simple and reliable way in Haskell**. So when your Haskell program errors with a  mysterious, 
rarely repeated error you might not learn much about it. There
is an effort to fix this once and for all and avoid many of the tradeoffs that now exist as far as backtraces in Haskell are concerned.
See [Decorate exceptions with backtraces](https://github.com/bgamari/ghc-proposals/blob/stacktraces/proposals/0000-exception-backtraces.rst). Also see
[Add backtraces to all exceptions](https://gitlab.haskell.org/ghc/ghc/-/merge_requests/6797). This important feature
is long delayed, but there is hope that the next major version of Haskell will have it.

OCaml gives you a nice backtrace if you set the environment variable as `OCAMLRUNPARAM=b`. You need to compile your
program with debugging information `-g`.

# OCaml Tooling in 2023

Much ink has been spilt on the inadequacies of OCaml tooling. I won't waste my time rehashing arguments in favor and
against `dune`, `opam` etc. dune & opam have come a long way in the last few years. If you've been burned in the past, it's
time to try again and see if OCaml tooling rises to your expectations. That's all I will say.

I will say that in 2023 dune's S-expression DSL is still not very pleasant (and difficult to remember) but **dune is 
wicked fast and feature rich** and gets the job done. I don't spend too much time on dune and once it's out of the way,
I concentrate on the OCaml program itself. I do need to spend more time in `dune` that I would in a `Cargo.toml` but 
I've made my peace with it. `dune` is not a deal-breaker for me.

I want to talk now about the LSP. OCaml's LSP in 2023 is **great**. If people tell you otherwise, it could be because they have
just not used it extensively in recent times. Like everything in OCaml, it's fast and responsive. Haskell's LSP feels a bit
more clunky and sluggish in comparison to me. OCaml LSP also work great on large projects. That's definitely been my
experience.

One advantage of the OCaml LSP over the Haskell LSP is that it supports goto definition outside your current project i.e.
goto definition/navigation in third-party libraries. If you use dune vendoring / opam-monorepo (which is very easy in most cases) 
then you can goto-definition outside the specific codebase you're working on. To me, this seems like a major limitation in Haskell. Right now there is even
a Haskell Summer project. See [Goto Definition for third-party libraries in HLS](https://summer.haskell.org/ideas.html#hls-goto-def-third-party) to remove this limitation.
BTW you don't need to do anything special with `rust-analyzer` to get goto-definition in third party libraries/rust core libraries.

One major weakness of the OCaml LSP (or you could say the compiler) it that it does not support searching for references
across a full code base (you can only see references within a single file which is basically useless). So it is currently not possible to point to a function and search where it is used in the whole codebase.
This is critical especially for OCaml because of the open-world assumption I spoke of above. Because membership of something
is not opt-in, you need to look at the existing codebase to see various usages of functions, modules, functor invocations etc. This is something that hopefully
change for the better. See the occurences [PR](https://github.com/ocaml/ocaml/pull/12142) that deals with this problem 
and will hopefully be complete and merged at some point soon.

# The anemic OCaml Runtime

The OCaml Runtime has also been accused of being anemic in the past. This is no longer true. OCaml code can run
parallely on multiple system threads now. This is the (new) OCaml "multicore" runtime available since
OCaml 5.0. 

Previously there was a "runtime lock" which essentially meant that only one system thread could run OCaml code
(though multiple system threads could be executing FFI C-code parallely, if you so wished). In that respect OCaml
resembled Python in some ways. Now OCaml is on par with Golang and Haskell runtimes.

There is another benefit: OCaml 5.0 supports effects which to a first approximation are basically "resumable exceptions".
Effects have been a very active area of research in type theory and runtime systems. Effects allow you to implement many
common facilities like lightweight green threads, various kinds of monads etc. in a very succint way.

Using effects we have a new library called [Eio](https://github.com/ocaml-multicore/eio) in OCaml. It allows IO in OCaml to look like IO in Golang. We can perform
a ordinarily blocking read call and the Eio will seamlessly pause the thread of execution and only
resume when data is available without you having to explicitly ask for that. All this happens without ugly and explicit
`await` calls or `>>=` invocations.

In OCaml 4.14 and below, this was done in a heavy-weight way by using libraries like Lwt or Async. With effects green
threads become easy. The beauty is that all this is accomplished by a library! The Ocaml runtime just provides the 
base functionality of effects (and OCaml domains) and Eio is able to build green threads using io uring on linux (or posix mechanisms like
`poll` when `io_uring` is not available).

In some ways, the OCaml runtime has become more advanced than the Golang runtime (and maybe the Haskell runtime). In order to provide new 
capabilities in the Haskell runtime you will (probably) need to write hard to maintain and fragile C-code. In OCaml, you
can, in most cases just use OCaml to enhance your custom "runtime". BTW the `Eio` library does not have a monopoly on effects and there can be other libraries that provide other approaches and benefits by using the same effect primitives.

And there is more to come. In a future OCaml version, effects will be tracked in the OCaml type system. Effects are
currently untyped. If you're interested in this area, Haskell 9.6 onwards has support for delimited continuations. 
See [here](https://github.com/ghc-proposals/ghc-proposals/blob/master/proposals/0313-delimited-continuation-primops.rst).
It provides support for effects/effect handlers like OCaml 5.0 does. It would be interesting to compare and contrast
but I don't know enough about it to comment further. My impression you could use it to build something like Eio in Haskell
but things are definitely not so far along as in OCaml.

# TL;DR Why should I use OCaml then?

> There are only two kinds of languages: the ones people complain about and the ones nobody uses.
> 
> Bjarne Stroustrup

OCaml appears in large number of places. The amount of OCaml being written is ever increasing. The complaints about OCaml, to me, say
that people are passionate about OCaml and want it to succeed. 

OCaml is only improving. However, in the meanwhile there is a lot that is right about OCaml that keeps
drawing me back and is worth recalling:

* The type system while nowhere near Haskell has a LOT to offer. It mostly fits in my head unlike Haskell
* The language has arguably better fundamental choices: no `undefined`, simpler exceptions (Haskell exceptions are vexing!)
* A simpler runtime: The haskell runtime has a ton of C-code!
* Lack of laziness (controversial opinion but this is my preference!)
* The compiler/type checker is super-fast and everything just feels responsive, lightweight and fast. I read somewhere
  that in terms of compilation speed/type system features ratio, it is probably best in class. I agree with that! 
  Think of the OCaml compiler being like the Golang compiler in speed but delivering so many more features!
* A very nice and consistent traditional class/object system in case you want that. "Objects/Classes done well". Newer anchor OCaml libraries like Eio  use objects/classes quite extensively
* Predictable, simple execution model
* Simple enough FFI. Building bindings with C libraries is nowhere as simple as Rust's bindgen but there are things like 
  [ppx_cstubs](https://github.com/fdopen/ppx_cstubs) which makes things very simple. Please note that it is simply
  not possible to be as good as Rust's bindgen because there is GC in OCaml which makes FFI a bit more complex.

OCaml can feel like a scripting language sometimes though everything is actually very strongly typed! There are many
places where OCaml could profitably replace Python. You could build mostly everything in OCaml and get great performance
and only FFI out when you absolutely need to. With Python, you would need to often write the module guts in C/Rust much earlier
if you want a high performance library.

Please try out OCaml. You might be pleasantly surprised.

