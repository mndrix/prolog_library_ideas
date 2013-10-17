# Standard Mode Indicators

```prolog
stuff(+A, -B) :-
    true.
```

into this:

```
stuff(A, B) :-
    must_be(A, ground),
    must_be(B, var),
    true.
```

Although modes are typically implicit in other aspects of a Prolog declaration, so I'm not sure how often these kinds of runtime assertions would be helpful.
