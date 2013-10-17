# Operating System Detection

A predicate `os(-OS:os) is multi` and another `is_os(+OS:os) is semidet` where os type includes at least the following:

  * `mac` (includes OS 9 and OS X variants)
  * `windows`
  * `darwin` (OS X only)
  * `linux`
  * `sunos`
  * `unix` (includes `darwin`, `linux`, `sunos`, etc)

`is_os/1` makes it easy to have code specific to certain groups without an nondeterminism.

For example, on my Desktop Mac running OS X 10.6.8:

```prolog
?- os(OS).
OS = mac ;
OS = darwin ;
OS = unix ;
false.
?- is_os(darwin).
true.
```

In SWI-Prolog, this can be built on top of `current_prolog_flag(arch, Arch)`.
