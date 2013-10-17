# Determinism Calling Assertions

## By the Caller

When calling a predicate, one might make certain assumptions about its determinism.  Sometimes I’d like to declare those assumptions to help future readers and to get runtime errors if the assumption proves wrong.  For example, in the following code `husband_wife(+A,-B) is det`.

```prolog
husband_wife(bob, jane).
husband_wife(tom, sue).
```

A caller working with that assumption might write one of the following (alternative syntaxes):

```prolog
call_det(husband_wife(bob,Wife)).
det husband_wife(bob, Wife).
husband_wife(bob, Wife) is det.
```

In each case, he declares his determinism expectation.  If the call fails or succeeds with a choicepoint, a helpful error is thrown during development.  Mercury’s determinism system is a good model to follow.

An implementation of `call_semidet/1` from the SWI-Prolog mailing list is:

```prolog
call_semidet(Goal) :-
    call_cleanup(Goal, Det=true),
    ( Det == true ->
        true
    ; % left choicepoints ->
        throw(error(mode_error(det,Goal),_))
    ).
```

## By the Developer

When developing a predicate, I might like to declare a predicate’s determinism and have a tool verify that my declaration is accurate.  It probably makes sense to extract determinism declarations from pldoc comments since the data is already there in a machine-readable format.  For example, the tool should fail this code because `husband_wife(bob,Wife)` has multiple answers:

```prolog
%% husband_wife(+Husband, -Wife) is det
husband_wife(bob, tina).
husband_wife(bob, sue).
husband_wife(tom, pam).
```

Using Determinism Calling Assertions (above), this could be implemented as a macro expanding to

```prolog
husband_wife(A, B) :-
    ground(A),
    var(B),
    !,
    call_det(husband_wife_aux(A,B)).
husband_wife_aux(bob, tina).
husband_wife_aux(bob, sue).
husband_wife_aux(tom, pam).
```
