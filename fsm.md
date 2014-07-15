# Finite State Machines

Make a small library for describing and working with finite state machines.  I want as little syntactic overhead as possible.  States are just atoms.  Transitions are just atoms.  Perhaps it looks something like this, with a single predicate defining a machine.

```prolog
%% light_switch(+Event, +CurrentState -NextState) is det.
%
%  Define a finite state machine named "light_switch".  This predicate defines
%  the states, the events and the transitions.
light_switch(flip, on, off).
light_switch(flip, off, on).
```
