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

Similar to the one in C-descended languages.  Syntax here borrowed from [Ciao](http://ciao-lang.org/docs/1.14/13646/CiaoDE-1.14.2-13646_ciao.html/fsyntax_doc.html). Convert this

```prolog
X = thing ? foo | bar.
```

into

```prolog
(thing -> A = foo ; A = bar),
X = A.
```

Be sure that cascaded ternary expressions are supported:

```prolog
fact(N, F) :-
   F = ( N = 0 ? 1
       | N > 0 ? N * fact(~ is N-1,~)
       ).
```

This construct is really an expression and in that sense it might fit well with `library(func)` which supports other types of expressions.

## Lexically Scoped Cleanup

One sometimes wants to acquire a resource (create a temporary file, set a flag, etc) and have that resource automatically released when a clause finishes.  Prolog provides `setup_call_cleanup/3` to help, but it has the following problems:

  * it’s ugly
  * it’s used with an extra indentation level or wrapper predicate
  * the setup and cleanup code can be arbitrarily far apart (depending on the size of the call argument)

I want something more like Go’s `defer` or Perl’s `Scope::Guard`, which keeps the clean up code lexically close to the acquisition code.  Compare this

```prolog
foo :-
    setup_call_cleanup(
        tmp_file(Tmp),
        ( bar(Tmp)
        , baz(Tmp)
        ),
        rm(Tmp)
     ).
```

with this

```prolog
foo :-
    tmp_file(Tmp),
    cleanup(rm(Tmp)),
    bar(Tmp),
    baz(Tmp).
```

A `term_expansion/2` macro would expand the latter into the former.  One should be able to call `cleanup/1` multiple times within a clause and have it work correctly.  The macro expansion should behave as if the following worked:

```prolog
term_expansion(
    (Head :- Setup, cleanup(Cleanup), Call),
    (Head :- setup_call_cleanup(Setup, Call, Cleanup))
).
```

With recursive expansion inside `Call`.

Prolog flags, because they are global state, should be managed automatically.  This suggests a predicate `cleanup_flag(+Flag, +NewValue)` which desugars into

```prolog
current_prolog_flag(Flag, OldValue),
set_prolog_flag(Flag, NewValue),
cleanup(set_prolog_flag(Flag, OldValue)),
```

### Prior Art

Jeff Rosewald, in `library(tipc)`, uses this:

```prolog
eventually_implies(P, Q) :-
      setup_call_cleanup(P, (Foo = true; Foo = false), assertion(Q)),
      Foo == true.

:- op(950, xfy, ~>).

~>(P, Q) :- eventually_implies(P, Q).
```

so, you can write code like this

```prolog
open(Something, read, In) ~> close(In),
do whatever you like,
!.  % cut reclaims resources
```

The syntax is really nice, but I don't like the way it uses cut.  I'm also hesitant to use a nice operator like `~>` for smoething that's done relatively rarely.

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

## Conditional Evaluation

I often write

```prolog
( Condition -> Goal ; true ).
```

maybe sugar it as

```prolog
if(Condition, Goal).
```

## Todo

`todo/0` and `todo/1` are helpful ways to mark a piece of code as not yet finished.  Inspired by Perl 6’s `…` operator and all the times I write `fail, % TODO` in Prolog.  Scala calls this `???`.

Executing it throws an exception.  In development mode this gives you a nice stack trace that you can use to understand the goal's context.  In production the exception catches your attention quickly so you know that you forgot to implement something.  I decided that an exception is better than failure because the former is easier to debug.  We don't want anything silent here.


## Membership

Declaring membership in a collection is a  common operation.  Witness how often one calls `member/2` or `memberchk/2`.  It might be nice if this operation were generalized and shortened.  The obvious candidate is an `in` operator like JavaScript, Python and C#.

```prolog
?- X in [1,2].
X = 1 ;
X = 2 .

?- howdy in greetings{hello:english, howdy:southern}.
true.

?- foreach(X in [1,2,3], writeln(hi)).
hi
hi
hi
true.
```

`in(?X, +Xs)` should call a multifile predicate `sweetner:has_member(+Xs, ?X)`.  Container libraries like assoc, ordsets, rbtree, etc. can add clauses to define what containment means.

Which semantics do we want: iterate all members, quit on the first match?  In other words, which is used more often: `member/2`, `memberchk/2`?  For some libraries (e.g., ordsets), the distinction is irrelevant because duplicate members are impossible.  Maybe iteration semantics is best since `once(X in Xs)` is clear and keeps membership separate from determinism.  If that becomes a common pattern, I can always have a macro that expands that construct into something more efficient that doesn't create choicepoints just to discard them right away.

Make sure that libraries can change their iteration behavior based on the placeholder term.  For example, iterating pairs seems natural:

```prolog
?- Key-Value in Assoc.
Key = a,
Value = 1 ;
Key = b,
Value = 2 .

?- Index-Value in [a,b].
Index = 1,
Value = a ;
Index = 2,
Value = b .
```

## Modules

### Syntax

The standard syntax for importing modules is verbose and repetitive:

```prolog
:- use_module(library(foo), [bar/3, baz/3]).
```

I'd like something shorter:

```prolog
:- use foo -> bar/3, baz/3.
```

I'd also like to be able to add and remove name prefixes from predicates:

```prolog
:- use uri -> uri_is_global/1, uri_encoded/3, without_prefix(uri_).
% imports is_global/1 and encoded/3

:- use foo -> hi/1, bye/2, with_prefix(foo_).
% imports foo_hi/1 and foo_bye/2

:- use julian -> form_time/{1,2}.
% imports form_time/1 and form_time/2.
```

These are strict syntactic changes which can be implemented with a small term_expansion/2 macro.

### Semantics

The current `use_module` systems allows a module to export predicates and operators.  I'd also like them to be able to export clauses (similar to multifile predicates) and perform arbitrary computations (change import paths, modify Prolog flags, etc).  I believe that all these goals can be realized by having `use foo, Args` behave as if it were

```prolog
:- load_files(library(func), [expand(true), if(not_loaded)]),
   func:export_to(CallingModule, Args).
```

(load module, transfer control to a predicate in the newly loaded module).  If the module doesn't implement `export_to/2`, we call a default implementation.  Otherwise, the module can do whatever it wants.  The default should work with all existing modules so that we don't lose access to all that code.

## All Solutions

It's quite common to find all solutions for a predicate that has only one "output".  The standard code is quite verbose, with variables floating around all over the place.  Perhaps we have something like this:

```prolog
all(foo, set(Foos)).
```

which is equivalent to

```prolog
all(Goal, Results) :-
  all_(Results, Goal).
all_(set(Results), Goal) :-
    setof(X, call(Goal, X), Results).
all_(bag(Results), Goal) :-
    bagof(X, call(Goal, X), Results).
```

## Deprecated

No longer part of the plan, but retained to help my memory.

### Short-circuting Disjunction

Even though it's no longer needed, I'm retaining this section to remind myself.  It should have been obvious that short-circuiting disjunction doesn't need any sugar.  This

```prolog
( summary(Entry, Summary) -> true
; content(Entry, Content), summarize(Content, Summary) -> true
; Summary=''
)
```

just becomes this

```prolog
once(  summary(Entry, Summary)
    ;  ( content(Entry, Content), summarize(Content, Summary) )
    ;  Summary=''
    ).
```
