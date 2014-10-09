# clp(pl)

A constraint logic programming library in which the constraints are arbitrary Prolog goals.  The library accumulates those goals in attributed variables.  It uses partial evaluation to simplify and specialize the constraints based on the current evaluation.  Constraint goals are reordered or executed to assist in the simplification.  Unifications are propagated through the constraints as they become available.  Labeling just executes the accumulated, simplified goal to generate solutions.


## Playground

A place to play around with some examples 

### Small list

For example, this code runs out of stack before finding the two valid answers.

```prolog
foo(L) :-
    length(L,N),
    N < 4,
    nth1(2,L,b).
?- findall(L,foo(L),Ls).
```

If `foo/1` is called with an unbound variable, we might make the following observations:

  * because `length/2` produces nonneg `N`, add goal `N >= 0` after `length/2` call
  * `N >= 0` and `N < 4` goals combine to `between(0,3,N)`
  * notice that `length/2` with two variables produces infinite solutions but `between(0,3,N)` produces only 4, move the `between/3` goal upwards (more deterministic goals should be as early as possible)
  * because `nth1(2,L,b)` produces a single solution, move it to the very top
  
So it's as if the code were originally:

```prolog
foo(L) :-
    nth1(2,L,b),
    between(0,3,N),
    length(L,N).
```

If we called `foo([A,B])` we'd make different observations:

  * propagate the binding for `L` through the goals
  * realize that `length([A,B],N)` can be executed to become `N=2`
  * realize that `N=2` is a unification and can be propagated through the goals
  * realize that `2<4` can be replaced with `true` which can be removed
  * realize that `nth1(2,[A,B],b)` can be executed to become `B=b`
  
So it's as if the code were originally:

```prolog
foo([_,b]).
```

In each case, we get the correct answer and I was able to write the Prolog code without regard to the order in which each goal (constraint) was executed.

#### Linear logic

I noticed an interesting use for linear logic in the simplifications above.  Assume that the body of a clause is a series of conjunctions of pure (no side effects) goals.  Then each goal can be viewed as a resource.  We have the following resources (assuming variables share across resources):

```prolog
length(L,N).
N < 4.
nth1(2,L,b).
```

Then the simplifications in the first example can be written as linear logic rules roughly like this:

```prolog
length(_,N) -o length(_,N) * N >= 0.
N>=L * N<U -o between(L,U,N).
```

From the second example, we have:

```prolog
length(L,N) * {nonvar(L)} -o { call(length(L,X)) }*  N=X.
X<Y * {number(X),number(Y),X<Y} -o true.
nth1(N,L,E) * {is_list(L),integer(N),N>0} -o {call(nth1(N,L,E))} * L=[...].
```
