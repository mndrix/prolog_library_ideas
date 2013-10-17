# Code Beautifier and Transformer

Similar to `Perl::Tidy` or `gofmt`, accept some Prolog code on standard input and print beautified Prolog in a consistent format to standard output.  It should apply popular formatting conventions, indenting conventions and whitespace conventions.  Tools like this are helpful on large projects to avoid conflict among developers over formatting style.  It also makes automated code transformation tools more effective (automated changes aren’t drowning in code formatting changes).

An automated code transformation tool could be a natural component of this (see `gofmt -r`).  It would essentially be a `term_expansion/2` or `goal_expansion/2` macro expander than operates on files when demanded instead of on code during every compile.
