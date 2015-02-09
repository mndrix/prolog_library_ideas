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

By displaying the examples in HTML documentation, users get a good idea how the predicate should be used.  A few accurate, well chosen examples are often more effective at communicating a predicate's purpose than paragraphs of natural language text.  Programmers are pretty good at exptrapolating a few examples into a mental model.

## Automated Tests

Each example can be executed as a goal.  If it succeeds, consider it a passing test.  Counter examples, when executed as goals, should fail.  This gives us assurance that our example-based documentation never deviates from the predicate's true behavior.

## Simplistic Type Inference

The predicate arguments shown in the examples allow us to perform primitive type inference.  In the examples above, the first argument has values `[]`, `[a,b]`, `[_]` and `[1,2,3]`.  By executing code like this, we can infer possible types:

```prolog
?- dif(T,impossible),
   foreach( member(V,[[],[_],[a,b],[1,2,3]])
          , error:has_type(T,V)
          ).
T = any ;
T = acyclic ;
T = nonvar ;
T = proper_list ;
T = list ;
T = list_or_partial_list ...
```

We can choose among the possible types with various heuristics.  Alternatively, the most precise type among the options is `proper_list`.  We can calculate "most precise" with something roughly like this:

```prolog
more_precise_than(T1, T2) :-
    forall( between(1,1000,_)
          , ( quickcheck:arbitrary(T1,Val)
            , error:has_type(T2,Val)
            )
          ),
    asserta(more_precise_than(T1,T2) :- !).
```

`more_precise_than/2` gives us a partial order.  By using a [topological sort](http://en.wikipedia.org/wiki/Topological_sorting) we can pick some types as the most precise.

### type_value/2

The predicates `error:has_type/2` and `quickcheck:arbitrary/2` suggest a more general relation: `type_value/2`.  It would have the following modes:

```prolog
type_value(+Type,+Value) is semidet.
type_value(+Type,-Value) is det.  % at random
type_value(-Type,+Value) is multi.
```

## Simplistic Mode Inference

A predicate's modes communicate three related details about argument instantiation and a predicate's solutions:

  * which arguments must start as `nonvar`
  * which initially `var` arguments will finish as `nonvar`
  * how many solutions are likely

By executing each example in every possible combination of var/nonvar arguments, we get a table like this:

```
length([],0)  1 step, 0 extra : ++ is det
length(_,_)   1 step, n extra : -- is multi
length([],_)  1 step, 0 extra : +- is det
length(_,0)   1 step, 0 extra : -+ is det.

length([a,b],2)  1 step, 0 extra : ++ is det
length(_,_)      3 step, n extra : -- is multi
length([a,b],_)  1 step, 0 extra : +- is det
length(_,2)      1 step, 0 extra : -+ is det.

length([_],1)  1 step, 0 extra : ++ is det
length(_,_)    2 step, n extra : -- is multi
length([_],_)  1 step, 0 extra : +- is det
length(_,1)    1 step, 0 extra : -+ is det.

% counter example
length([1,2,3],7)  1 step, 0 extra : ++ is semidet
length(_,_)        7 step, n extra : -- is multi
length([1,2,3],_)  1 step, 0 extra : +- is det
length(_,7)        1 step, 0 extra : -+ is det.
```

For a predicate of arity N, there are 2^N possible modes.  Each mode for which all examples and counterexamples behave as expected is a legitimate mode for this predicate.  For each mode, choose the determinism with the most solutions.  For length/2, that leads to the following inferred modes:

```prolog
length(+,+) is semidet.
length(+,-) is det.
length(-,+) is det.
length(-,-) is multi.
```

If one of these modes is incorrect, the user can provide an example or counter example to show the discrepancy.  The inference algorithm can then come to the right conclusion.

## Steadfastness Tests

["Steadfastness basically means that you cannot force a predicate down the wrong path by filling in output arguments wrongly"](http://permalink.gmane.com/gmane.comp.lang.ai.prolog.swi/9785).  Based on the inferred modes, fill in some output arguments "wrongly" (by giving them a `nonvar` value) and make sure the example still works.
