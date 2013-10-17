# Whitespace Sensitive Syntax

I personally like Prolog’s syntax.  It has a good mix of consistency and flexibility.  Unfortunately, people love to hate it.  And the comma vs full-stop problem is obnoxious in version control tools.  I think it would be fairly easy to implement a whitespace sensitive syntax as a strict superset of Prolog syntax which desugars to normal Prolog.

This code

```prolog
append([], A, A).
append([A|B], C, [A|D]) :-
    append(B, C, D).

% naive reverse
reverse([],[]).
reverse([X|Xs], Zs) :-
    reverse(Xs, Ys),
    append(Ys, [X], Zs).
```

might be written like this

```
append [] A A
append [A|B] C [A|D]
    append B C D

% naive reverse
reverse [] []
reverse [X|Xs] Zs
reverse Xs Ys
    append Ys [X] Zs
```

We’re basically inferring parentheses, commas, full stop and `:-` using rules like this:

  * line starting in column 0 is a clause head
  * adjacent lines starting in column 0 imply a full stop ending the first line
  * line indented more than the one before implies :- ending the first line
  * adjacent lines starting beyond column 0 imply a comma ending first line
  * line indented less that the one before implies full stop ending the first line
  * first atom in a line is a functor. remaining tokens are arguments

I'm not sure how disjunctions should look.

