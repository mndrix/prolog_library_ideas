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

There's probably also a `struct/2` which is `struct/3` with `[]` as the final argument.  It describes default structs.  There's also a `struct_dict(Struct, Dict)` for converting between structures and built in dicts.

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

There's also `Property/3` which is a shortcut for `Property/4` where the "old" value is ignored.

```prolog
?- height(building(300,E),500,Building).
Building = building(500,E).
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

## Mapping, Folding and DCGs

I've encountered some circumstances where it would be helpful to transform one struct into another via a predicate `map_struct/3`.  Imagine a struct representing a person.  If she marries, her name and marital status both change.  Instead of doing:

```prolog
marriage(Woman0,HusbandName,Woman) :-
    surname(Woman0,_MaidenName,jones,Woman1),
    marital_status(Woman1,single,married,Woman).
```

we might want something like:

```prolog
marriage(Woman0,HusbandName,Woman) :-
    map_struct(marry(HusbandName), Woman0, Woman).

marry(HusbandSurname,surname,_MaidenName,HusbandSurname).
marry(_HusbandSurname,marital_status,single,married).
```

If the higher-order predicate fails, it leaves that field's original value in place.

If I changed the argument order on `Property/4` to `foo(OldValue,NewValue,OldStruct,NewStruct)` we could also do something like:

```
marriage(Woman0,HusbandName,Woman) :-
    foldl( call
         , [ surname(_,HusbandName)
           , marital_status(single,married)
           ]
         , Woman0
         , Woman
         ).
```

or using a DCG with adjusted argument order:

```
marriage(HusbandName) -->
    surname(_,HusbandName),
    marital_status(single,married).
```

## Reflection

Each struct is identified by a unique name.  We don't allow structs with the same name and different arity because that makes it difficult for structs to acquire new fields later.  We have the following predicates for reflecting on currently defined structs:

```prolog
%% current_struct(+Name) is semidet.
%% current_struct(-Name) is nondet.
%
%  True if a struct with Name has been defined.  Used for iterating and testing the existence
%  of structs.

%% struct_property(?Name, ?Property).
%
% True when Name refers to a struct with property Property.  Name can also be a struct value, in which
% case that term's functor is used as the name.
%
% Property is one of:
%
%   * field(Field)
%     True if the struct has a field named Field.
%   * field_type(Field,Type)
%     True if the struct has a field named Field whose type is Type.  If the field
%     has no explicit type, it's given as =|any|=.
%   * field_default(Field, Default)
%     True if the struct has a field named Field whose default value is Default.
%     If the field has no explicit default, this property fails.
```

## Visibility

library(record) creates terms and predicates that are visible only within a specific module.  The arrangement described above assumes that structs are globally visible.  Is global visibility really what we want?  I can see benefits of creating both public and private structs.  Implementing private structs, given the UI and behavior I want, seems quite complicated.  I'm also not sure if I want the verbosity of having users choose visibility when they declare the struct.  That path leads to Java's "public static void ..." keyword soup.

I need to think about this some more.  In general, local declarations are much better than global ones.
