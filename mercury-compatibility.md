# Mercury Compatibility

It would be cool if one could write a single program which runs in both Prolog and Mercury.  One would have to adopt a subset of Prolog (without `!/0`, for example), but Prolog macros should be able to implement most of the `:- type` and `:- func` annotations.

This would let one use all the cool static analysis tools that Mercury has.  Although … perhaps a better approach would be to just implement similar static analysis tools for Prolog.
