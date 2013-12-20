# Documentation Mode Lines

SWI Prolog documents predicates with a mode line to indicate:

  * a predicate's determinism
  * each argument's type
  * each argument's mode
  
Typical mode lines look something like this:

```prolog
length(+List:list, -Length:integer) is det.
length(-List:list, +Length:integer) is det.
```

The stanard library `pldoc/doc_modes` parses these declarations to perform validations, generate HTML documentation, etc.  Unfortunately that library provides no helpful API for parsing, generating or working with these mode lines.  The suggestions in this document could either be an outline for refactoring that standard library or could be released as a separate library.  A quick look at the existing code makes me lean towards the latter.

## Purpose

The goal is to provide a data type which cleanly represents these mode lines.  One should be able to parse a single mode line into this data structure, serialize it back and get the exact same thing we started with.  There would be a bunch of helper predicates for working with these structures.

## Structure of Mode Lines

Consider this mode line:

```prolog
foo(?A:list(T), -X:T)// is multi
```

Here is the name for each part:

  * `foo` - predicate name
  * `?` and `-` - argument mode indicators
  * `list(T)` and `T` - argument types
  * `//` - slashes (to indicate a DCG)
  * `multi` - predicate determinism

There should be predicates working with each of those components.

We might represent this mode line with the following value (using library(maybe) for optional values):

```prolog
modeline( foo  % predicate name
        , 2    % predicate arity
        , args( arg( just(?)
                   , A
                   , just(list(T))
                   )
              , arg( just(-)
                   , X
                   , just(T)
                   )
              )
        , just(multi)  % determinism
        ).
````

## String Description

library(pldoc/doc_modes) treats a mode line as a Prolog term which allows it to use standard read and write predicates for input and output.  I think that makes the code more complex, fragile and redundant than it needs to be.  I'd prefer to describe a mode line's textual form with a DCG.  That single description would let us parse and generate textual mode lines.

The only challenge I foresee with a DCG is that we must take care to make sure that `list(T)` and `T` share the same variable.  Type inferencers will want to unify `T` with a concrete type when they discover one.
