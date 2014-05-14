# Smartmatch Operator

Perl 6 has a smartmatch operator whose purpose is to match a data structure against a pattern and returning a boolean result.  I’ve found it to be a useful abstraction.  It might be nice to have something similar in Prolog.  It would succeed if a term matches a pattern.  The main interface would be:

```prolog
~~(+Value, +Pattern) is semidet
```

True if `Value` matches `Pattern`.  The precise semantics of pattern matching depend on the definition of multifile predicate `smartmatch/2`.

One of the key design goals is to allow libraries to extend this operator to do something reasonable for their problem domain.  See Perl’s smartmatch table for inspiration.  Here are some possible examples/extensions:

  * `X ~~ List` becomes `memberchk(X, List)`
  * `X ~~ Assoc` becomes `get_assoc(X, Assoc, _)`
  * `X ~~ [Key=Value|Pairs]` becomes `memberchk(X=_,[...])`
  * `X ~~ type(Type)` becomes `error:has_type(Type, X)`

The Perl community has learned [many valuable lessons](http://markmail.org/message/u22titrq5ljclz4b) about this operator in the 4 years since they implemented it.  Read that thread for the problems and a thorough discussion of how they plan to resolve them.  Summary: the default smartmatch table needs to have only 4-5 entries, with a clear extension mechanism.

We don't want too much DWIM.  Some of these problems are solved because Prolog is strongly typed.
