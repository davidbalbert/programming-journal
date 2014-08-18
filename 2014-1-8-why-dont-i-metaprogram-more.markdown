# Why don't I metaprogram more?

I'm working on adding tail call optimization to my most recent Lisp
interpreter. The first step is to add a proper AST to encode whether or not an
expression is in the tail position. If you don't encode this information, by
the time you eval your expression, it will already be too late. Whether or not
an expression is a tail call depends on the context surrounding it, something
you don't know if you're eval function operates on s-expressions.

S-expressions get parsed into an AST after macroexpansion but before
evaluation. That way macros still operate on lists (otherwise it wouldn't be
much of a Lisp), but eval gets the extra information encoded in the AST.

Which is how we get to the question of metaprogramming. My AST has one node
type for each special form, one for function calls, and eventually, one for
tail calls. With the exception of field names, each special form node is pretty
much the same. It has to know things like its name, its arg count, whether it
takes a variable number of arguments, and how to parse its data out of a list.
My first naive implementation looked something like this:

```ruby
class Def < AST
  attr_reader :var, :value

  def initialize(args)
    check_arg_count(args)
    @var = args[0]
    @value = args[1]
  end

  def name
    "def"
  end

  def argc
    2
  end

  def varargs?
    false
  end
end
```

I knew that I could do better with some metaprogramming, so I decided to give
it a shot. Here's the interface I came up with:

```ruby
class Def < AST
  name "def"
  args :var, :value
end
```

For vararg forms:

```ruby
class Lambda < AST
  name "lambda"
  varargs :arglist, :bodies
end
```

Pretty nice! Here's the implementation:

```ruby
class AST
  def self.name(name = nil)
    if name
      @name = name
    else
      @name
    end
  end

  def self.args(*arg_names)
    attr_reader *arg_names

    define_method :argc do
      arg_names.size
    end

    define_method :parse_args do |args|
      arg_names.each do |name|
        instance_variable_set("@#{name}", AST.build(args.car))
        args = args.cdr
      end
    end
  end
end
```

Every time I write code like this in Ruby, I think "Wow, that's complicated! Is
it really worth it?" There is lots of Ruby that looks like this, and to great
effect. Active Record does some pretty neat things with tricks like these.

Metaprogramming in Ruby is complicated because Ruby doesn't have macros. Ruby
gives you two options for metaprogramming. The worse of the two is `eval` and
string interpolation. It does have some good properties, namely that your
strings look a lot like normal Ruby, but you lose things like syntax
highlighting, and if you're not careful, you could open yourself up to security
issues that are similar to SQL injection attacks but have the possibility of
remote code execution. Yuck.

The second option is through reflection (an imperfect name for it, but I
couldn't think of a better one). In Ruby, anything that can be done with
keywords can also be done with method calls. This includes making new classes,
modules, methods, etc. This makes the language super powerful (at least
approaching the power of a language with macros), and it lets you do all sorts
of metaprogramming magic including my AST definitions. Unfortunately this
tends to be a lot more verbose than eval with string interpolation and doesn't
look nearly as much like the code you would have written.

Macros are awesome and I wish Ruby had them. Specifically what I want is the
equivalent of quoting and unquoting syntax. Those are some pretty powerful
ideas. Lisp macros are super nice because you already know the exact shape of
the data structures that represent AST nodes. They're lists! I think macros are
a little less elegant in langugaes that have syntax, because you need to know
the field names for the structs that represent all the syntax. That said,
quoting and unquoting get you most of the way there because you don't have to
manipulate the expression objects directly.

Short version? Macros are cool. I'd metaprogram more if Ruby had macros.
