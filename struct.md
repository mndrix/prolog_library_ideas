# Structured data

`library(record)` is very handy.  It gives a convenient syntax for creating structured data types and related predicates for working with them.  Unfortunately, the predicates it creates are verbosely named, confusingly named (I always have to read the documentation to remember), prevent more abstract predicates from taking advantage of similarities across records, and prevent some declarative code which Prolog might use to advantage.

I want a library which fills the same niche but avoids these shortfalls.  It provides `library(struct/record)` which is a strict superset of `library(record)`.  It does everything that `library(struct)` does but also supports complete backward compatibility with `library(record)`.

## Conventions

In the text below, assume the following type declarations:

```prolog
has_type(pair, _Key=_Value).
```

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

There's probably also a `struct/2` which is `struct/3` with `[]` as the final argument.  It describes default structs.

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

There's also a semidet, two-arity predicate for each property that just declares the existence of a property on a struct.

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
