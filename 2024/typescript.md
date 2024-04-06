# Typescript

Published April 6, 2024.

Let's assume you are the maintainer of a JavaScript[^1] library, which provides type
definitions in TypeScript for a variety of reasons that are out of the scope of
this thought experiment. You have a function that a user of your library will
call with some sort of configuration. As happens often with JavaScript, you have
functions that can take fairly polymorphic data, and your type definitions
eventually end up blocking cases that you (and your users) know are valid. This
isn't great. Your users are now unable to write perfectly valid code without
disabling the type checker, which isn't a pleasant experience.

[^1]: While the broader philosophical point I want to make doesn't rely on
    specifics about the language, the motivating factor for me writing this post
    was some slagging on TypeScript, and I wanted to steel man that.

<details>
<summary>An aside on performance.</summary>

Wow, I think this is the shortest I've gone before having to go on an aside.
Regardless, if you are facing this issue in your own code, you should know that
having a polymorphic function can affect your performance, especially if said
function is on the hot path. Details of that are beyond the scope of this post,
but [this
post](https://romgrk.com/posts/optimizing-javascript#2-avoid-different-shapes)
has a benchmark that shows just how bad it can be.

For the rest of this post, I'm assuming the performance isn't terribly relevant.
The function we're talking about may not on the hot path (e.g. configuration) or
your functional requirements dictate what you can optimize. If that isn't the
case, the rest of this post doesn't make any sense: before thinking about types,
make the function less polymorphic. You might not need to do anything else.

</details>

Now, you could remove or radically simplify your types to perform the bare
minimum of checks, and replace some of your type checks with runtime assertions.
You document this thoroughly and explain to your users that they should ideally
write tests so that these runtime assertions don't cause production failures.
This is a perfectly valid approach, I have no qualms with it, and if you chose
to do this I fully support you and the rest of this post is in no way criticism
of you or your choice.

But there is another choice. You could reach for the more complex features of
TypeScript to write types that cover all valid inputs without allowing invalid
inputs. This usually requires considerable time and effort, and your types will
often be inscrutable at first glance. Of course, you liberally comment your new
type definition both for yourself and any future maintainers, and in general,
help people understand what's going on.

<details>
<summary>An aside on TypeScript syntax</summary>

Many might seize upon my admission that the type would look inscrutable at first
glance, and will assert that that is the biggest argument against such types. I
don't agree. 

TypeScript's syntax for types *is* terse and unfamiliar to most, and requires
some learning before you can read and understand types in one go.  For example,
the use of `?` instead of `if` is probably the biggest reason why most people
find it difficult to parse. I personally found the way the `[]` operator works
difficult to read too. 

However, I think you should learn how to read the language you are using! I
believe most people would be able to pick up the TypeScript type syntax with
some effort. It certainly is less alien than APL or factor, and I see plenty of
people who can learn and use those productively.

</details>

I, personally, am a believer in this approach. Of course, the type you are
writing is complex, and potentially hard to read, especially for beginners to
TypeScript. This adds a non-trivial maintenance burden on yourself and any
future maintainer. But on the other hand, your users (which could include you!)
get a lot more out of it. Since they're already using TypeScript (otherwise, why
would they care?) they will now *always* know in advance if their function call
will work or not, no runtime assertions, no tests to write or maintain. And,
most importantly, in most cases they will not even have to see or understand the
type definition: they will get an error telling them whats wrong when they mess
up [^2], and they can fix that error to never have to think about the type
again. This is a big deal, at least in my opinion.

[^2]: I will admit that TypeScript errors are not great, especially for more
      complex types, but I would still prefer them over no error until runtime.

I believe this choice is not popular among some sectors of programmers. A recent
post explaining a complex TypeScript type on lobste.rs was met with derision.
But I believe there is a strange aversion to complexity in types, which often
doesn't exist for other kinds of complexity. For example, many of the same
people on lobste.rs will lap up articles about complex data structures and
systems used for things as simple as a text editor, but will look skeptically on
a complex type. Before you dismiss the possibility of using a complex type,
consider if you'd be open to adding as much complexity that is hidden from your
users if it wasn't something type related.

And while I hope I've convinced you that there's merits to this approach, I also
want to explain why I'm in favor of it philosophically. Years ago, when I was a
young developer cutting his teeth on programming, a colleague impressed upon me
the importance of the essential but complex force multiplying work that allows
the rest of us to use safe, ergonomic APIs that help us be productive. My
colleague's example was Ruby: the language is notoriously complex to parse, all
to, as DHH[^3] puts it:

> make the machine appear to smile at and flatter its human co-conspirator

[^3]: Yes, I understand the irony of quoting DHH on a post defending typescript,
    but I will take my joys where I can :^)

I think this is a beautiful thing, and if I can write a slightly convoluted
looking type that can save many users a little anguish when their code fails in
production, I am glad to do so. Therefore, I think this choice, too, is an
eminently reasonable one.
