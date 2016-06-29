# Command line option parsing

SWI-Prolog has at least two ways to process command line options:

  * [library(optparse)](http://www.swi-prolog.org/pldoc/doc/swi/library/optparse.pl)
  * [argv_options/3](http://www.swi-prolog.org/pldoc/doc_for?object=http_unix_daemon%3Aargv_options/3)
 
The first library has the user define a schema for the command line options and also generates automated documentation.  The second library just converts `--name=value` pairs into `name(value)` terms.  Both libraries make the parsed options available as a list.  Lists are convenient, but difficult to query and reason with.

My ideal command-line parsing library would have the following features:

  * convert command-line arguments into predicates in an `argv` module (or whatever module the user prefers)
  * allow, but not require, an argument schema
  * work with an arbitrary list of atoms (not just the program's argv)

For example, a command line like this

```
-v --size=27 --follow-redirects
```

might create a module like this

```prolog
:- module(argv, []).

v.

size(27).

follow_redirects.
```

which allows one to write code like:

```prolog
main :-
    warn("starting program"),
    ( argv:size(Size), Size<99 -> true; Size=42 ),
    format("You asked for size=%d", [Size]).

warn(String) :-
    ( argv:v -> writeln(String); true ).
```
