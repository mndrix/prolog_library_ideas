# Syntactic Sweetener

Some syntactic sugar ideas are so small that they don’t really justify a library of their own. It’s probably best to roll them all into `library(sweet)` which I can use across projects.  It’d be great if `library(sweet)` re-exported the behavior of `library(func)`.

## Argument Aliases

I often want to write a predicate which refers to a compound term and many of its parts.  I currently write that like:

```prolog
foo(x(A,_,C,D)) :-
    X = x(A,_,C,D),
    % use X
```

I want to be able to write it like this (similar to Haskell):

```prolog
foo(x(A,_,C,D)@X) :-
    % use X
```

Alan Baljeu [asked for this on the mailing list too](https://lists.iai.uni-bonn.de/pipermail/swi-prolog/2013/012013.html).  He wanted SWI Prolog to add special indexing, but syntactic sugar could accopmlish the same goal without much trouble.


## Ternary Operator

Just like the one in C-descended languages.  Convert this

```prolog
X = thing ? foo : bar.
```

into

```prolog
(thing -> A = foo ; A = bar),
X = A.
```

For compatibility with Ciao, consider using `?` and `|` instead.

This construct is really an expression and in that sense it might fit well with `library(func)` which supports other types of expressions.

## Short-circuting Disjunction

I often find myself writing code like this:

```prolog
( summary(Entry, Summary) -> true
; content(Entry, Content), summarize(Content, Summary) -> true
; Summary=''
)
```

Namely, trying several alternatives to bind a variable and committing to the first one that succeeds.  I'd rather write something like:

```prolog
(  summary(Entry, Summary)
or ( content(Entry, Content), summarize(Content, Summary) )
or Summary=''
).
```

As best I can tell, the only widely accepted notation for short-circuiting disjunction is `||`, but that's an invalid operator in Prolog.

## Lexically Scoped Cleanup

One sometimes wants to acquire a resource (create a temporary file, set a flag, etc) and have that resource automatically released when a clause finishes.  Prolog provides `setup_call_cleanup/3` to help, but it has the following problems:

  * it’s ugly
  * it’s used with an extra indentation level or wrapper predicate
  * the setup and cleanup code can be arbitrarily far apart (depending on the size of the call argument)

I want something more like Go’s `defer` or Perl’s `Scope::Guard`, which keeps the clean up code lexically close to the acquisition code.  Compare this

```prolog
foo :-
    call_cleanup( ( tmp_file(Tmp)
                  , bar(Tmp)
                  , baz(Tmp)
                  )
                , rm(Tmp)
     ).
```

with this

```prolog
foo :-
    tmp_file(Tmp),
    defer(rm(Tmp)),
    bar(Tmp),
    baz(Tmp).
```

A `term_expansion/2` macro would expand the latter into the former.  One should be able to call `defer/1` multiple times within a clause and have it work as if one called `defer((ignore(Goal1),ignore(Goal2))`.

Prolog flags, because they are global state, should be managed automatically.  This suggests a predicate `defer_flag(+Key, +NewValue)` which desugars into

```prolog
current_prolog_flag(Key, OldValue),
set_prolog_flag(Key, NewValue),
defer(set_prolog_flag(Key, OldValue)),
```


## Otherwise

Make `otherwise/0` an alias for `true/0`.  It reads nicely in lengthy if-then-else constructs by maintaining symmetry.

```prolog
( foo(X) ->
    do_foo_stuff
; bar(X) ->
    do_bar_stuff
; otherwise ->
    do_default_stuff
).
```


## Todo

`todo/0` and `todo/1` are helpful ways to mark a piece of code as not yet finished.  Inspired by Perl 6’s `…` operator and all the times I write `fail, % TODO` in Prolog.

It behaves just like `fail/0`.  In development mode, it calls `print_message/2` first.  `todo/1` lets the user specify a reason why the code is left undone.
