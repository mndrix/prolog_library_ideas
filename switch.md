# Switch Statement

```prolog
...,
match X with [
    7 -> writeln(elway),
    42 -> writeln(answer),
    (alpha; beta) -> writeln(greek_letter),
    between(1,6) => writeln(bar)
].
```

becomes

```prolog
compiled_switch_1234(7) :-
    !,
    writeln(elway).
compiled_switch_1234(42) :-
    !,
    writeln(answer).
compiled_switch_1234(A) :-
    ( (A=alpha;A=beta) -> writeln(greek_letter)
    ; between(1,6,A) -> writeln(bar)
    ; fail
    ).

...,
compiled_switch_1234(X).
```

Consider desugaring to something that uses the [smartmatch operator](smartmatch.md) so that one can match against patterns supplied by libraries.  Also consider [Python’s switch proposal](http://www.python.org/dev/peps/pep-3103/) for ideas.

Ideally, switch cases where we’re just doing unification would be optimized by automatically creating an intermediate predicate.  That gives us really good performance by leveraging Prolog’s first term index.

If cuts inside the switch statement are broken, generate a compile time error saying so.  That way it doesn’t surprise users.
