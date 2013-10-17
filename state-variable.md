# State Variable Syntax

Similar to Mercury's state variables.  Convert something like this:

```prolog
foo(List!, List!) :-
    one(List!, stuff, List!),
    two(List!, List!).
```

into this

```prolog
foo(List0, List2) :-
    one(List0, stuff, List1),
    two(List1, List2).
```

It just assigns new variables in a sequency from top to bottom, left to right.
