# Infer IO Usage

Infer whether a predicate does IO (prints to screen, opens a file, sleeps, etc).  All foreign predicates are considered to do IO (except for a whitelisted group of exceptions).  This whitelist must be hardcoded as facts in the library.  For other predicates, they perform IO if one of their subgoals performs IO.  This should be an easy inference rule to implement recursively.

This can be useful for static analysis.  It could also be useful for running untrusted Prolog code.  It’s not perfect since one can build atoms from arbitrary data and then execute those atoms.  To be truly safe, the underlying Prolog engine must provide something like `call_without_io(...)` but this inference could cover many cases.

`library(sandbox)` does something similar.  Investigate there before writing any code.
