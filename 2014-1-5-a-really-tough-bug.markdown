# A really though bug

I've been writing a store passing Lisp interpreter, and I ran into a really
tough bug. Here are two pieces of basically equivalent code:

```lisp
(def second (comp car cdr))
(map second '((a 1) (b 2)))

(map (comp car cdr) '((a 1) (b 2)))
```

The former worked without any problems, but the latter crashed with the error
message "((a 1) (b 2)) is not callable."

Here's what comp looks like:

```lisp
(defun comp (f g)
   (lambda (& args)
      (f (apply g args))))
```

After a bit of poking around, I found that `((a 1) (b 2)`, which I'm going to
call "the list" from now on, was being bound to one of the arguments to `comp`,
and thus called by the anonymous function that `comp` was returning. I knew
that `comp` wasn't the problem because it worked fine in the first example.[^1]
A bit more debugging revealed that `comp`'s arguments were bound correctly when
the anonymous function was created, but were bound incorrectly when it was
called.

This left three possibilities.

1. I was mutating the environment hashes between creating and calling the
   anonymous function. This seemed unlikely because my whole purpose for
writing a store passing interpreter was to only use immutable data structures.
That said, Ruby's hashes are mutable, even if I chose not to change them, so it
was possible that I was doing so accidentally.

2. I was reusing store locations, shadowing the values of `f` and `g` with the
   arguments to `map`. This would point to a bug in my code that generates the
next store location. If this were true, I'd be able to see multiple instances
of locations 69 and 70 (`f` and `g`) in the store at the time `map` was called.
The newer values would be incorrect and the older ones would be correct.

3. I was not threading the store through my program properly. At some point in
   the program, I would be passing an old version of the store to `eval` and
losing a handle to the original nodes 69 and 70. When the arguments to map were
bound, the next available location would once again be 69 and when the
anonymous function was eventually called, it's arguments would be the same as
the arguments to `map`.

I was able to rule out the first option quickly. The environment hashes are not
used often, so it was easy to look at all the places they're used and verify
that I'm not mutating them.

The second option took a bit more work to eliminate, though I think I made it
harder than I needed to. At the time map was called, I was able to assert that
the store had only one value with location 69 and location 70, which meant I
was not shadowing the proper values of `f` and `g`.

This left option 3. I looked in every place where a new store was created, and
looked at every place I called one of those methods, and I could find nothing
amiss, yet it was clear that I had lost two store nodes by the time `map` was
called. I resorted to opening a debugger session and aimlessly moving up and
down stack frames to see if I could find anything. Miraculously, I did.

The code I had been looking for was of the form:

```ruby
eval_without_macroexpand(...)
```

That's a method on `Interpreter`. Almost all instances of that method call
should have had an explicit receiver to thread the new store through the
program:

```ruby
interp.eval_without_macroexpand(...)
```

None of the calls were missing the explicit receiver. What I found in my
aimless stack exploring was the following code.

```ruby
expr.car.call(self, *evaled_args)
```

This is a function call. Self is an instance of `Interpreter`. This was where I
forgot to thread the interpreter through. It should have been:

```ruby
expr.car.call(interp, *evaled_args)
```

I felt stupid and relieved.

The most frustrating part of all this was that even afterwards, I couldn't
figure out a good strategy for debugging this issue other than staring at my
code until I found the problem, which is rarely effective. The good news is
that I just came up with one!

My assumption was that `eval_without_macroexpand` was being called only once
per interpreter object. This was wrong. But, because this is Ruby and my data
structures aren't actually immutable, there's an easy way to verify this. Set a
`already_evald` flag on each interpreter object when `eval_without_macroexpand`
is called. At the beginning of the method, throw an exception if the flag is
already set. This will reveal the interpreter that is being double eval'd and a
quick trip up the stack would reveal the logic error. Victory!

[^1]: It took me a frustratingly long time to believe this. I had to rewrite
comp in a few different ways before I was convinced. Not believing what is
obviously true is one of my weaknesses as a programmer. I end up making changes
and running tests without having hypotheses to back them up. Nick calls it
superstitious programming, and I think that's a good name for it.
