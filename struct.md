# Structured data

`library(record)` is very handy.  It gives a convenient syntax for creating structured data types and related predicates for working with them.  Unfortunately, the predicates it creates are verbosely named, confusingly named (I always have to read the documentation to remember), prevent abstract predicates from taking advantage of similarities across records, and prevent some declarative code which Prolog might use to advantage.

I want a library which fills the same niche but avoids these shortfalls.  It provides `library(struct/record)` which is a strict superset of `library(record)`.  It does everything that `library(struct)` does but also supports complete backward compatibility with `library(record)`.

## Examples

Structured data types are declared exactly like `library(record)` but using the `struct` operator.

```prolog
:- struct building(height:integer, erected:integer).
:- struct mountain(height:integer, country:atom).
```

I don't foresee any extensions here.  The current practice is quite good.

One creates a new struct value using `struct/3`

```prolog
?- struct(Building, building, [height=300, erected=1983]).
Building = building(300, 1983).
?- struct(Thing, Type, [height=300]).
Thing = building(300, _),
Type = building ;
Thing = mountain(300, _),
Type = mountain .
?- struct(mountain(_,_), Type, _).
Type = mountain.
```

There's probably also a `struct/2` which is `struct/3` with `[]` as the final argument.  It describes default structs.  Or maybe `struct/2` is reserved for `struct(Struct, Dict)` for converting between structures and built in dicts.

Each property has a predicate describing the relationship between a property's value and a struct containing that property.

```prolog
?- height(building(300,1983), Height).
Height = 300.
?- height(Thing, 300).
Thing = building(300, _) ;
Thing = mountain(300, _) .
?- height(Mountain, 300), country(Mountain, nepal).
Mountain = mountain(300, nepal).
```

This convention works well with `library(func)` because each property relation can be used as a function:

```prolog
?- format('The building is ~d meters high~n', [height $ Building]).
The building is 300 meters high
```

There's also a semidet, unary predicate for each property that just declares the existence of a property on a struct.

```prolog
?- height(M), country(M).
M = mountain(_,_).
```

It's quite common for property names to be known only at runtime.  One can use `property/3` and `property/2` in that case.

```prolog
?- property(building(300,1983), height, Val).
Val = 300.
?- property(Struct, location).
Struct = mountain(_,_).
```

One changes properties with `Property/4` or `property/5`

```prolog
?- height(building(300,E),300,500,Building).
Building = building(500, E).
?- property(mountain(N,pakistan), pakistan, location, india, Mountain).
Mountain = mountain(N,india).
```

## Polymorphism

Using the same predicate for each property of the same name allows one to have predicates that work across structs:

```prolog
taller_than(Short,Tall) :-
    height(Short, X),
    height(Tall, Y),
    X < Y.
?- taller_than(building(20,_), mountain(1000,_)).
true.
```

## Default Values

I like the way that `library(record)` allows, but doesn't require, default values.  When a value is not supplied and not defaulted, it's left as a variable.  This matches well with Prolog's notion that variables represent values which aren't known yet.  It may be tempting to be more strict here (require Go-style "zero values" or mandate a default for each field, but I think that not the Prolog way).

## Auto loading

The design described above poses one challenge.  The `struct` module doesn't know the name of all struct elements at the time `library(struct)` is loaded.  That means it can't export accessor predicates yet.  By defining `exception/3`, our library can hook itself into the autoloading mechanism.  When a predicate is called (at runtime) but not found in the relevant module, we can look for a matching predicate in the `struct` module (which is fully defined by this time).  If one matches, assert a proper predicate definition in the calling module and have the autoloader try again.

## Types

`library(record)` allows one to specify a type for each field.  These types are convenient for documentation and development.  I'd like `library(mavis)` to handle all the typing for a struct's implementation.  This approach has been working well in my Quietude project.

A field's type can also be a helpful way to resolve ambiguity.  In some cases (integer vs atom), backtracking can resolve the ambiguity as long as relevant predicates don't throw exceptions.  In cases where backtracking won't work, is there a way to communicate type information to explicitly resolve ambiguity?  For example,

```prolog
:- struct particle(flavor:quantum_flavor).
:- struct ice_cream(flavor:culinary_flavor).
```

In this case, both `quantum_flavor` and `culinary_flavor` are likely subtypes of `atom`.  Backtracking may not always resolve the ambiguity between them.  What if we allowed variants on the accessors like this:

```prolog
?- flavor(Thing, Flavor, quantum_flavor).
Thing = particle(Flavor).

?- flavor(Thing, Flavor, culinary_flavor).
Thing = ice_creame(Flavor).

?- flavor(Thing:quantum_flavor).
Thing = particle(_).
```

`flavor/3` needs an extra argument because `Flavor` could reasonably contain a `:/2` term.  However, `flavor/1` can get by because a struct will never have `:` as its name.  Unfortunately, that notation for `flavor/1` makes it look like we're describing the type of `Thing` rather than the type of its field.

I need to think about this some more.  Although, perhaps in the real world it's so rare for a field to have the same name and different types and be involved in the same program that it's not worth special treatment.  It's easy enough to follow `flavor/2` with a type check, that that probably suffices.
