# Example-driven Development

Prolog is well-suited to what I think of as example-driven development.  A predicate represents a true relation among some variables. By giving examples and counter examples, we get:

  * documentation
  * simple automated tests
  * simplistic type inference
  * simplistic mode inference
  * steadfastness tests

## Examples

Let's consider an implementation for `length/2`:

```prolog
length(L,N) :-
    length(L,0,N).
    
length([],N,N).
length([_|L],N0,N) :-
    succ(N0,N1),
    length(L,N1,N).
```

We might declare examples like this:

```prolog
:- examples
    length([],0),
    length([a,b],2),
    length([_],1).
:- counter_examples
    length([1,2,3],7).
```

## Documentation

By displaying the examples in HTML documentation, users get a good idea how the predicate should be used.  A few accurate, well chosen examples is often more effective at communicating a predicate's purpose than paragraphs of natural language description.  Programmers are pretty good at exptrapolating a mental model based on a few examples.

## Automated Tests

Each example can be executed as a goal.  If it succeeds, consider it a passing test.  Counter examples, when executed as goals, should fail.  This gives us assurance that the documentation we provide to the user accurately reflects the predicates behavior.

## Simplistic Type Inference

The predicate arguments shown in the examples allow us to perform primitive type inference.  In the examples above, the first argument has values `[]`, `[a,b]`, `[_]` and `[1,2,3].  By executing code like this, we can infer possible types:

```prolog
?- dif(T,impossible),
   error:has_type(T,[]),
   error:has_type(T,[a,b]),
   error:has_type(T,[_]),
   error:has_type(T,[1,2,3]).
T = any ;
T = acyclic ;
T = nonvar ;
T = proper_list ;
T = list ;
T = list_or_partial_list ...
```

We can choose among the types using various heuristics.  Alternatively, the most precise type among the options is `proper_list`.  We can calculate "most precise" with something roughly like this:

```prolog
more_precise_than(T1, T2) :-
    forall( between(1,1000,_)
          , ( quickcheck:arbitrary(T1,Val)
            , error:has_type(T2,Val)
            )
          ),
    asserta(more_precise_than(T1,T2) :- !).
```

## Simplistic Mode Inference

TODO

## Steadfastness Tests

TODO
