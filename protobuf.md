# Protocol Buffers Sugar

SWI-Prolog has a nice Protocol Buffers library.  Unfortunately, the protocol definitions are done directly in Prolog rather than using the nice, native protobuf syntax.  Write a DCG that parses and generates Protocol Buffer files.

Then write a tool compatible with `protoc` (named `protoc-gen-prolog`, I think) which compiles Protocol Buffers descriptions into Prolog files.  That way, the standard description files can be used across languages.
