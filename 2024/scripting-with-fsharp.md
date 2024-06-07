# My Experiences Scripting with F#

I'm a big believer in using scripts to automate my regular, personal computing.
In fact, I've moved a lot of things I used to do in apps to regular files I
store on a homeserver and scripts that operate on those. This system works
*really* well for me, and removes almost all of my frustrations with siloed
apps. [^1] Naturally, I have a lot of scripts!

[^1]: I won't pretend it doesn't *create* other frustrations, but at least I can
    write code to deal with them!

One of the advantages of scripting here is that I can write them in any
language. The bulk of my scripts are in Ruby, my "first love" in programming,
but I also extensively use JS (via Deno) and Python where it makes more sense.
As a functional programming enthusiast I wanted to explore using an ML-like
language for scripting. I chose F# after evaluating a few languages for this
purpose. My first choice was OCaml, but its anemic standard library made it hard
to use for scripting. F# comes with the stacked .NET standard library, which I
felt like would keep me from having to use dependencies. And unlike Haskell, I
like F#'s simple pragmatism towards IO and mutability.

This wasn't my first rodeo with F#. I've written a couple of simple servers with
Giraffe and done a few advent of code problems with it. I also work with Rust in
my day job, and have written a non-trivial amount of OCaml, so I come with some
experience in the ML-like language space. However, my dislike of F#'s OO side
had kept it from being my preference over OCaml and Rust when writing
applications. Ideally, my scripts would be simple enough to not need OO at all,
which would eliminate the biggest problem I had with F#. The other advantages
were pretty compelling over the competition, so I was excited to try it out.

It has been about a year since I started scripting in F#, and now I have just
under a thousand lines running the gamut of things that one normally does with
scripts: filesystem access, network requests and a fair amount of glue logic. To
commemorate this, I thought I'd write down my experiences with scripting in F#
in the familiar "The Good, The Bad, The Ugly" format.

## The Good

### I <3 ML (and so, F#)

There's just something about ML languages that match the way I think about code,
and that makes writing F# a breeze. I usually start with modeling the data I
need and how it should flow through the script, and F# makes that simple with
all the ML goodies. In particular:

- Pipelines are a great fit for scripting, which frequently comes down to "do
  this thing that can fail and then the next". I wrote a custom "result
  pipeline" operator to do this with fallible functions.
- The type system is flexible with anonymous records and variants, a big plus
  over Rust. For parts of the script where I didn't care too much about the
  model I could skip a lot of boilerplate.
- Since I was writing solely for myself, I experimented with a few of the more
  complex language features. I was very happy to find a great use case of active
  patterns!

### F# is super simple to deploy

The .NET runtime ships with `fsi`, which is all I need, so my scripts can be
`.fsx` files with a shebang up top. Admittedly this isn't a big deal, and most
languages will have something like this, but for a compiled language that uses
bulky .NET project files I wasn't expecting that.

But wait: fsi also has the sweet sweet sweet `#r` syntax, which is
*significantly* better than many, if not most, languages. No other ML(-ish)
language has this (AFAIK). Even in "scripting" languages, Python still doesn't
have this[^2], and Ruby and Elixir require a bunch of boilerplate. I'd say `#r` is
my second favourite version of this behind Deno's URLs, which allow you to
reference packages which are not on a package manager.

[^2]: Technically you *can* call pip programmatically, but that isn't on the
    same tier of support as inline bundler, programmatic use of Mix or `#r`.

I'll admit that I haven't used this system enough to know the gory details of
where it can go wrong, but for my scripts I've had no issues with version
conflicts, and they've happily kept chugging so far.

### Scaling up is easy

It's really, really simple to refactor your scripts, split them into multiple
files, use modules to encapsulate, etc. This is significantly easier than with
Ruby because the type checking helps avoid mistakes when I refactor, and I'm a
lot more confident with a complex script in F# than with Ruby. I still hate that
you can't have cross-referencing definitions without using `and`, but for
scripts this is relatively rare.

## The Bad

### The Standard Library 

Don't get me wrong: the stdlib is great... for applications. It's just not good
for scripting concisely.

One of my first roadblocks was `System.Net.Http`. If you want to make a network
request without leaning on a NuGet package, this is your best bet. The problem
is that *it is very verbose.* I didn't care about async, since my scripts were
only making a few network requests, so I had to:

- Set up a an `HttpClient`
- Create a `HttpRequestMessage`
- Read the content from the `HttpResponseMessage`

This is a lot of boilerplate! If I wanted to use one of the convenience methods
`HttpClient` provides, I'd have to use Async, and that also leads to a bunch of
boilerplate. One of the reasons I wanted to use a functional language is that
the expressivity reduces pointless boilerplate, but the Standard Library forces
me back into it?

In contrast, in Ruby I can just:

```rb
Net::HTTP.get("https://example.org")
```

This even scales up beautifully: I can POST, add query params, add a body
without having to deal with boilerplate. 99% of my use cases for network
requests can be handled with 1-2 lines. Meanwhile, the *bulk* of my cases with
F# involve writing far too much code. Given how many of my scripts dealt with
making network requests, I quickly gave up on using the standard library and
used Fsharp.Data, which kind of made one of the biggest reasons I chose to use
F# moot.

Likewise, I absolutely love that Python's standard library ships with SQLite by
default, and it is disappointing that I can't do that with .NET.

### OO is inevitable

This ties in with the Standard Library comment above. Both with the standard
library and with packages you import from NuGet, it is very hard to avoid having
to use OO. In scripting, you don't need to worry too much about writing your own
classes or interfaces, but my biggest problem with OO is that it doesn't compose
as well, especially with pipelines, and requires a lot more boilerplate. [^3]

For example, if you're using an OO-structured library, you often have to create
objects to pass to it. If you're lucky, there's a builder or a fluent API for
these, which makes it simpler, if not as convenient or composable as pipelining.
In other cases it's just worse, and you have `<-`s everywhere to set fields. I
wish there was at least something like Kotlin's `apply` to make this easier.

[^3]: F# 8.0 made this better with the `_` shorthand, but I wrote much of my F#
    scripts before that and I haven't had reason to write more F# scripts since
    8.0 came out. Also, I think `_` is a bandaid over the real problem of
    impedance mismatch, and it's not as clean as using a purely functional
    system. F# could really benefit with something like
    [UFCS](https://tour.dlang.org/tour/en/gems/uniform-function-call-syntax-ufcs).

## The Ugly

### Exceptions everywhere

I *hate hate hate* having to deal with exceptions from the standard library and
NuGet packages. Exceptions break the "soundness" of the type system, meaning
everything I like about ML ends up being less and less useful. They don't
compose well, and intermixing a function that throws exceptions with functions
that return results means writing and wrapping lots of boilerplate. 

This was the single biggest reason I scaled down my usage of F#. I'd write some
code, notice that it didn't return a `Result` even thought it was fallible, sigh
and write boilerplate to convert the exception into a result. Even worse was
when I used a library and it threw an exception in runtime that I wasn't
expecting. I was very tired of the boilerplate explosion, and with all the
`try/with`s my scripts were becoming as cumbersome as Ruby or Python.

## Conclusion

As I mentioned, I ended up scaling down usage of F# for new scripts, usually in
favour of Deno and Ruby. If it's something simple, I use ruby, and if it's more
complex I use Deno with TS. I should stress that this doesn't mean that F# is
*bad*. In my opinion, Deno[^4] with typescript is the most ergonomic environment
for scripting, with all the advantages of F# and few disadvantages. And I say
that despite JS/TS being a poor match for my aesthetics. F# compares really
favourably to Deno, but the impedance mismatch between the language's aesthetics
and the broader ecosystem it is in hurts me more than writing JS. Admittedly
some of the F# 8 changes have caught my eye and made me excited to try it again
for the more complex scripts I have on the horizon.


[^4]: I haven't yet tried Bun, but I think it could be even better because my
    biggest issues with Deno was Node library compatibility. Bun's scripting
    syntax for shell usage also looks really sweet.

