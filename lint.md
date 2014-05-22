# Lint Tool

Make a lint-like tool for spotting suspicious Prolog constructs.  See examples section below.

According to [this thread](http://computer-programming-forum.com/55-prolog/514e80369d8e92ea.htm), a tool called nit is available as part of [NU-Prolog](http://ww2.cs.mu.oz.au/~lee/src/nuprolog/).  Perhaps it could be ported to SWI-Prolog or ideas taken from it.

There’s an existing SWI-Prolog linting tool called [BLint](http://www.fing.edu.uy/~gbrown/prolog/blint.html).  It could probably be packaged up as a pack and made usable.

This could probably be implemented via `library(prolog_codewalk)`.  By loading the target code into memory, walking every clause, reporting on suspicious patterns.

I’d love to integrate this tool into SWI-Prolog’s `make/0` command.  It already does undefined predicate checks and runs tests.  It’d be a helpful place to report on other suspicious constructs.

In an unrelated email thread, Richard A. O’Keefe said,

> The thing I dream of for Prolog is something like PLT Scheme (now Racket) or clang --analyze, something that helps me write *working* code.

That really is the emphasis of this lint tool.  Help developers get from where they are to working code, without having to write and run a bunch of tests.

Erlang’s success typing is another great place to look.  It only warns of problems when it can prove that you did something wrong.  If it can’t prove it, it stays silent.

It’s worth considering whether the lint tool should work at the syntactic level or at the code level (aka “post-macro” level).  SWI-Prolog’s semantic singletons already do some good analysis at the code or compiler level.  Maybe there’s space for a syntactic linter.  We could allow library authors to define additional lint rules based on correct usage for their library.  For example, it’s typically an error to call `delay/1` before `!/0` in a clause.  The linter shouldn’t have to hard-code that rule, but because of macro expansion it’s hard to detect it after macros have been expanded.  Performing checks at the syntactic level makes both aspects easier, I think.

## Examples

Some small examples:

  * mismatch between `format/2` template and number of arguments
  * variables in a term with suspiciously similar names (sans trailing numbers)
  * report on undefined predicates
  * ! in multifile predicates

### `forall/2` loop variables

The first argument of `forall/2` must have at least one variable that's local to the forall/3.  For example, this is almost certainly mistake eventhough the goal succeeds:

```prolog
N = 3,
forall( between(1,7,N), writeln(N) )
```

The developer probably intended:

```prolog
N = 3,
forall( between(1,7,I), writeln(I) )
```
