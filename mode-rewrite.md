# Rewrite clauses to accommodate all modes

Imagine the following useful predicate:

```prolog
%% json_dict(+Json:atom, -Dict:dict)
%% json_dict(-Json:atom, +Dict:dict)
%
%  True if Json is a JSON representation of Dict.
json_dict(JsonAtom, Dict) :-
    atom_json_term(JsonAtom, json(EqPairs), []),
    maplist(eq_dash, EqPairs, Pairs),
    dict_pairs(Dict, _, Pairs).
`

Reading that predicate with a purely declarative semantics suggests that it should work in the following two modes:

  * json_dict(+, -)
  * json_dict(-, +)
  
However, to realize both modes, one must write two clauses, with the goals reordered.  One must also check the arguments to see which mode applies and cut to help the indexer.  All of that is repetitive grunt work.  I wish I could somehow declare that the order of goals within this clause is irrelevant and have the compiler generate the necessary permutations to support all modes described in the documentation.

If one has data about each goal's allowed modes, implementation should be easy enough:

  * split clause body into a list of goals
  * sort goals by mode (acceptable modes first, etc)
  * generate mode testing code to choose which goal order is desired for a given invocation
  
Mercury does this automatically based on its mode system.  I'm not quite sure how to declare the modes in Prolog.  A modeline in documentation typically means, "This predicate supports these modes" and not "rewrite this predicate to support these modes".
