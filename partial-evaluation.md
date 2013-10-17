# Partial Evaluator

My [library(term_util)](https://github.com/mndrix/term_util) does some code simplification. That can be used when building the full, partial evaluator.

The paper _An Automatic Partial Evaluator for Full Prolog_ by Dan Sahlin, March 1991, presents a useful system for partially evaluating Prolog code.  At a high level, it iterates a few simple rules against a piece of code to produce a new piece of code.  I’d love to have something like that available in a library.

I imagine a predicate `partial_eval(Goal, NewGoal)` which performs the partial evaluation and produces a new goal which can replace the original.  An important step in partial evaluation is simplification of Prolog goals.  library(term_util) handles a bunch of that already.

Some goals can be evaluated at compile time, thus further simplifying the code and making way for further simplifications.  Considering how many clauses have rudimentary type checks before a cut, compile time evaluation could open many further simplification possibilities.

These simplification rules, along with the partial evaluation rules, can be applied to a piece of code until we reach a fixed point.  Then we have the simplest version of the code that we can obtain.

If the partial evaluator has access to a mode/purity inference tool, it could perform a bunch of additional simplifications.  For instance, a !/0 goal can swap places with an immediately preceding, deterministic goal.  See Sahlin’s paper (section 3.3) for his classification of predicates into three purity groups: _side effect_, _propagation sensitive_, _logical_.

This purity and mode information could fit nicely with `library(delay)` mode annotations.  For example, we might have the following:

```prolog
% delay:mode(+Modes, +Determinism, +Purity)
mode(system:atom_codes(ground, _), semidet, pure).
mode(system:atom_codes(_, ground), semidet, pure).
mode(system:atom_codes(var, var), error, io).
```

`library(delay)` would ignore those modes with `error` determinism since its purpose is to avoid that scenario.
