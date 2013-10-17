# DCG Utilities

SWI-Prolog comes with `library(dcg_basics)` which includes some convenient helpers for writing DCGs. Make a module which reexports all those predicates and adds a bunch more helpers:

  * `padded_integer(Padding:code, Width:integer, N:integer)`
    * integer N with character Padding added to the left to make a total Width
  * `repeat(DCG, N:integer)`
    * exactly N copies of the pattern DCG
  * `star(DCG)`
    * zero or more instances of DCG, greedy
  * `plus(DCG)`
    * one or more instances of DCG, greedy
  * `split(Separator:dcg, Parts:list)`
    * split at each separator to find the parts. Elements consumed lazily for the parts
  * `anywhere(DCG)`
    * match DCG anywhere inside a string
    * `anywhere(DCG) --> string, DCG`

Search through all my Prolog code for `-->` to find examples of helpful DCGs that should be in this library.

Be sure that each predicate has a thorough test suite.
