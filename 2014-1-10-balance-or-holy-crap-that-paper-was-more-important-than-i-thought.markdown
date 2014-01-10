# Balance, or Holy crap that paper was more important than I thought

Ever since I read the objects vs abstract data types paper, I feel like all my
programming possibilities have doubled. Where I saw one way to do things, I now
see two. I wasn't programming exclusively in one of the modes before. It was a
mix of both, but now I know which was which. Consider these two examples of
`eval` in a Lisp interpreter (that's all it seems I can write these days).
They both assume an AST like I touched on in my last post.

Abstract Data Types:

```ruby
def eval(expr, env)
  case expr
  when Def
    ...
  when If
    ...
  when Lambda
    ...
  # etc
  end
end
```

Objects:

```ruby
class Def
  def eval(env)
    ...
  end
end

class If
  def eval(env)
    ...
  end
end

class Lambda
  def eval(env)
    ...
  end
end

# etc
```

Wow! Neither example is particularly revolutionary (the abstract data type
example is probably more common as an implementation of `eval`), but for me,
the ability identify which is which and to be able to see one and imagine the
other is huge.

What I think is most important is not this realization, nor this particular
paper, but timing. Two weeks ago, reading that paper, or any other good paper,
was probably near the top of the list of things that would help me become a
better programmer. On the other handIf I were to only read papers I doubt I
would become much better at all. There are all sorts of things I can do to
become better. I can write code, read papers, take classes (I'm in the Coursera
crypto class right now!), do code review, get code review, etc. Figuring out
the right balance is one of the things I struggle with most and the programmers
I admire seem to have.
