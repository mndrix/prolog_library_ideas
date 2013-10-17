# Type Inference

It can be helpful for documentation and debugging and compile-time error detection to be able to infer the types of variables in goals.  Prolog has a great system for describing types based on `error:has_type/2`.  This predicate simply recognizes values which belong to the type in question.  Types are defined as the conjunction and disjunction of Prolog goals (a boolean algebra) and boolean algebra is a distributive lattice.  That means we can learn types and join them with other learned types to obtain a supertype.

This library has a predicate something like `infer_types(+Goal, -Types:list(pair))`.  Running the predicate on a goal gives you a list of variables and their associated types.  The types can be named (like `atom`) or compound (like `integer(N), 0 =:= N mod 2`).  For example,

```prolog
atom_length(Atom, Length) :-
    atom_codes(Atom, Codes),
    length(Codes, Length).
?- infer_types(atom_length(A,N), Ts).
Ts = [A=atom, N=integer].
```

Calculating the type of a conjunction is calculating the types of variables in the first goal and the types of variables in the second goal.  Then join those two type inferences to produce the type inference of the conjunction.

We could start with simple boolean algebra join rules (assume `v` is join) such that

```prolog
nonvar(X) v atom(X)
```

is

```
nonvar(X),
atom(X).
```

but later specialize the join rules so that it’s just `atom(X)`.  That’s because an atom is a more specialized case of nonvar.
