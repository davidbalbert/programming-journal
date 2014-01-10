# My first paper

I read "On understanding data abstraction, revisited" by William R. Cook
(http://www.cs.utexas.edu/~wcook/Drafts/2009/essay.pdf). This is the second
paper I've read, and the first one I've finished.[^1] Here is a quick summary
that is hopefully comprehensible and will verify that I understood the paper.

People often say that abstract data types and objects are the same, but they're
not. ADTs and objects are both forms of data abstraction. Data abstraction is a
concept referring to any technique that allows you to hide the implementation
details of data from someone using your data.

ADTs use type abstraction to hide data, while objects use procedural
abstraction. In a nutshell, ADTs have a public name and a hidden
representation, while objects have a public interface (a set of methods they
respond to) and private implementations of those methods. An object's methods
can only see it's own representation, not the internal representation of
another object. It must interact with other objects using methods defined on
their interfaces. This applies even for other objects that implement the same
interface. Functions defined on an abstract data type can see the internal
representation of any number of arguments of that data type.

OOP makes it easy to add new, independent representations of data. All you have
to do is implement the same interface. For instance, you can imagine making a
`FakeFile` that acts just like a `File` but doesn't actually persist anything
to the filesystem. Someone [did this](https://github.com/defunkt/fakefs) in
Ruby to make FileSystem testing easier. As long as your code doesn't every
check that you have a thing of type `File`, you're in the clear. Using objects
can also make your code harder to reason about. What will happen when you call
`f.write("Hello, world\n")`? Who knows! It depends on whether you have a `File`
or a `FakeFile` or a `Socket` or something completely different. This is hard
to do with ADTs because you can't make a new representation of an ADT without
knowing about all the other internal representations of the type. For example,
if `File` was an ADT that was implemented by some internal representation
called `RealFile`, only the creator of the `File` ADT would be able to make a
`FakeFile` because she would have to change all of the operations defined on
`File` to be aware of `FakeFile` in addition to `RealFile`.

Using ADTs makes it easy to define new operations on your data. This is because
you know the complete internal representation and don't have to deal with
unforeseen other representations in the future. It also allows you to better
optimize binary and multi-ary operations by inspecting the representation of
all of the arguments of your type rather than just one argument (generally
`self` or `this` in in OOP). It's hard to add new operations with  objects. To
use an example from the paper: imagine you have a Set interface that includes
the methods `isEmpty()`, `contains(i)`, `insert(i)` and `union(otherSet)`. It
would be impossible to add an `intersect(otherSet)` method without changing the
interface because we don't have a way to iterate over all items in `otherSet`.
You could add a method to the interface that lets you iterate over the sets,
but that wouldn't work for infinite sets, something our interface currently
supports. It's easy to add an `intersect` function with an ADT representing a
set, because we know the full set of possible internal representations of our
data. This comes with a loss of flexibility. Maybe our internal representation
has precluded us from making infinite sets. Too bad.

Choice quote:

> One conclusion you could draw from this analysis is that the untyped
> λ-calculus was the first object-oriented language.

I'm stoked that I kind of understand what this means. I'm not going to try an
explain it. Go read the paper.

[^1]: The first paper I read but didn't finish was "Scheme: An Interpreter for
Extended Lambda Calculus" by Guy Steele and Gerald Sussman, which is the paper
that introduced the Scheme programming language. I lost steam in the last
section which is an implementation of a Scheme interpreter written in MacLISP,
an old dialect of Lisp that I don't know well and found hard to understand not
the least because it doesn't even have lexical scope, which of course was
introduced to the Lisp family by Scheme :).
