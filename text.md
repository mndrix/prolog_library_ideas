# Generic string library

I want a generic, consistent interface to string data.  The library should provide an interface across all SWI-Prolog's different string representations:

  * atoms
  * lists of codes
  * strings

Start by implementing the interface for these representations. Then implement it for other text data structures.

## Predicates

The library should have the following predicates (and many more):  

  * `is_text(+Text) is semidet` - true if Text a valid text representation
  * `text_atom(Text:text,Atom:atom)`
  * `text_codes(Text:text,Codes:list(nonneg))`
  * `text_string(Text:text,String:string)`
  * `text_length(+Text:text, -Length:nonneg)`
  * ...
  
  
## Typeclasses, interfaces, multifile predicates, oh my!

The predicates defined in this library are all `multifile` so that third-party libraries can add their own string-like data structures.  Many of these predicates can have default implementations that are implemented in terms of others.  For example, we might have something hypothetical like:

```prolog
text_length(Text,Length) :-
    text_codes(Text,Codes),
    length(Codes,Length).
```

An implementation of this interface should be able to implement `text_length/2` directly or implement the dependencies and rely on the default implementation.  This is similar to the way that Haskell typeclasses work and it's really handy.

Prolog doesn't have a well-defined convention for typeclass-like constructs.  The closest thing is `multifile` predicates, but that requires each implementation to be aware of dangling choicepoints, etc.  I'd like something more declarative:

  * a typeclass can define which predicates are part of its interface
  * a module can declare
    * that a specific value meets that interface
    * which predicates on that value implement which interface methods
  * the typeclass module chooses an implementation based on these declarations and the default implementations
  
  
## String representations

Fill this section with data structures that would make good string implementations in Prolog.  [Ropes](https://en.wikipedia.org/wiki/Rope_(data_structure)) come to mind, but I don't know much about them.
