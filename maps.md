# Associative Arrays (Maps)

`library(assoc)` and `library(rbtrees)` are a good implementations, but I don't like the interface and they're too rigidly tied to a specific implementation.  Prolog has no panacea for representing maps.  Some programs use associative lists (`[a-1, b-2, …]`).  Others use something like `library(assoc)` provided by their implementation.  Others use SWI-Prolog dicts.  Changing approaches requires substantial modifications to one’s code.

I want a generic map interface and several implementations.  Changing an import statement should be all that’s needed to switch implementations.  (In my fantasy world, asking for a generic map statically analyzes how the map is used and chooses an implementation that works well in those conditions.)

Each implementation, in this library or an external library, exports predicates with the same names and semantics. Loading a new library is all that’s needed to get better map performance.

Operations that change the number of key-value pairs are free to upgrade/downgrade to a different implementation as appropriate.  For example, a map implemented as a sorted list of key-value pairs, upon adding a 30th pair, might yield a map that’s implemented as a tree or a skip list instead.  That way, an implementation can defer to a better implementation if that becomes preferable.

A fundamental predicate is `map_delta(?MapWithout,?Key,?Value,?MapWith)`. It can be used to add a new pair, remove an existing pair, or calculate which pairs are missing between two other maps.  There may be more specific predicates `insert/3`, `delete/3`, etc. which are faster, but a library that provides a single `map_delta/4` gets free implementations for the rest.

## Declarative maps

The maps described above must be persistent.  It's also valuable to have [declarative maps](https://github.com/mndrix/ddata).
