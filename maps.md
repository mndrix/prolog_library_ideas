# Associative Arrays (Maps)

`library(assoc)` and `library(rbtrees)` are a good implementations, but I don't like the interface and they're too rigidly tied to a specific implementation.  Prolog has no native way to represent maps.  Some programs use associative lists (`[a-1, b-2, …]`).  Others use something like `library(assoc)` provided by their implementation.  Changing approaches requires substantial modifications to one’s code.

I want a generic map interface and several implementations.  Changing a goal (at map creation) is all that’s needed to switch implementations.  In my fantasy world, asking for a generic map statically analyzes how the map is used and chooses an implementation that works well in those conditions.

At the highest level, one creates a map with `new_map(Map)`.  It makes no guarantees about the implementation and might choose one at compile time based on static analysis.  Curlies might be macro sugar for creating maps: `Map = {}`.  One can create a new, non-empty map with `new_map([a-1,b-2], Map)` or `Map = {a=1, b=2}`.

There’s also sugar for choosing an implementation based on expected size and operations: `new_map([size(100), 1000*lookup/3, delete/3, size/3], Map)`.  From that, the library chooses an implementation which minimizes the expected cost.

These high level constructs desugar to

```prolog
:- multifile new_map/4
new_map(+Implementation:atom, -Map:map).
```

Each implementation, in this library or an external library, defines `new_map/2`, all operations it supports and `map_cost(+Implementation, +Operation, -Cost)`.  All these predicates are multifile so that loading a new library is all that’s needed to get better map performance.

Operations that change the number of key-value pairs are free to upgrade/downgrade to a different implementation as appropriate.  For example, a map implemented as a sorted list of key-value pairs, upon adding a 30th pair, might yield a map that’s implemented as a tree or a skip list instead.  That way, an implementation can defer to a better implementation if that becomes preferable.

A fundamental predicate is `map_delta(?MapWithout,?Key,?Value,?MapWith)`. It can be used to add a new pair, remove an existing pair, or calculate which pairs are missing between two other maps.  There may be more specific predicates `insert/3`, `delete/3`, etc. which are faster, but a library that provides a single `map_delta/4` gets free implementations for the rest.
