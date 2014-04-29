# Prolog Web Framework

Writing a web server in SWI Prolog requires a lot of repetitive, low level coding.  I'd like something that's more declarative.

## Use Cases

A collection of use cases:

  * dispatch requests to predicates based on unification
    * based on request method (`get`, `post`, `head`, etc).
    * based on path (`foo/Bar/baz`, notice the `Bar` variable)
  * easy access to request details (path, parameters, form, url, etc)
  * unambiguous access to request details (request as list creates ambiguity between request components)
  * support middleware to factor out common layers
    * request logging
    * proxy header handling
    * memory limits
    * request time limits
    * converting request body into a term based on `Content-Type` header
    * converting request parameters into terms based on the parameter name
  * built on a generic web server interface like
    * [PEP33](http://www.python.org/dev/peps/pep-0333)
    * [Rack](http://rack.rubyforge.org/doc/SPEC.html)
    * [JSGI](http://jackjs.org/jsgi-spec.html)
    * [PSGI](http://search.cpan.org/~miyagawa/PSGI/PSGI.pod)

## HTTP Request library

At the base of all these ideas is a library for parsing and representing HTTP requests.  By providing a single, common representation of an HTTP request, web servers and middleware have a lingua franca allowing them to interoperate.

### Atoms or Strings?

One must choose whether to represent textual content as an atom or as a string.  We use an atom if the textual content is drawn from a "small" set of possible values.  Otherwise, we use a string.

So HTTP methods are atoms (there are only a few of them).  HTTP paths components are atoms (an application may have 20-30 unique path components).  HTTP header names and URL parameter names are also atoms (there may be a couple hundred of them at most).

If in doubt, use a string.

### Parsing raw HTTP requests

The first step is to parse a raw sequence of bytes (stream, `list(code)`, string, atom, iterator or enumerator) into a barely-structured term.  The stream of bytes should probably be represented by extending `library(pure_input)` to work with byte sequences other than streams (perhaps by converting them to in-memory streams at first).  That way, we can use DCG rules to handle the parsing.

The "barely-structured" representation is something like this (`ll` for "low-level"):

```prolog
request_ll( Method : atom
          , Path : string
          , Query : maybe(string)
          , Header : string
          , Body : string
          ).
```

A `Body` can nearly always fit into memory.  For really huge bodies, we can create a string that's backed by a mmap'd file.

The lingua franca representation is a term with more structure and greater flexibility on types.  Something like:

```prolog
request( Method : atom
       , Path : list(atom)
       , QueryParams : multimap(atom, any)
       , Header : multimap(atom, any)
       , Body : any
       ).
```

The speculative `multimap` type maps a single key to 0 or more values.  Predicates for accessing a key's value iterates all possible values on backtracking.

Here are some notes about each of those fields:

  * `Method` - lowercase atom like `get`, `post`, `delete`, etc.
  * `Path`
    * original path split on `/` characters
    * first, empty path component removed
    * list structure allows easy unification like `Path = [user, UserId, name]`
  * `QueryParams`
    * values are of `any` type
    * middleware can "inflate" a value from `string` (original type) into a type with more structure
  * `Header`
    * same rationale as `QueryParams`
  * `Body` - middleware may inflate to a type with more structure

## Middleware

I imagine that all middleware acts as if it were one giant DCG:

```prolog
apply_middleware(Request0, Request) -->
    first_middleware,
    second_middleware,
    third_middleware,
    ...
    final_middleware,
    !,
    unify_current_state_with(Request).
```

Each middleware may leave choicepoints and they remain until we finally find a request structure that can satisfy them all.  This allows a given middleware to support multiple formats for its results.  For example, imagine a middleware which converts a `Date` header into one of: `get_time/1` format, `stamp_date_time/3` format, `library(julian)` format.  The `Date` middleware can perform all three conversions on backtracking.  A downstream middleware just acts as if the value were in the proper type.  If that assumption is wrong, backtracking fixes it.

For performance, the first middleware could `freeze/2` a bunch of type assertions.  That way we get quick failure and avoid backtracking over a bunch of work.  I suspect that good conventions will develop and backtracking will happen fairly rarely, but it'll be really valuable when it's needed.

## Hooks

There can be hooks (probably handled by a middleware) for converting unstructured headers and parameters into structured ones.  For example,

```prolog
header_value(date, Raw, D) :- parse_time(rfc_1123, Raw, D).
header_value(date, Raw, D) :- form_time(rfc_1123(Raw), D).
header_value(content_type, Raw, Mime) :- parse_mimetype(Raw, Mime).
...
header_value(_, Raw, raw(Raw)).
```

Or maybe the backtracking on these hooks generates multiple values for a given key in the `Header` part of the request.  A consumer can say `get_header(foo, Val), float(Val)` to indicate that they want a floating point representation of that header.

## Routing

An essential part of web service programming is routing inbound requests to the piece of code that should handle that request.  Comments above hint at this idea but I need to specify it in more detail.  Fundamentally, this is a predicate mapping a request value to a closure handling that request.

Consider ideas from `pack(arouter)` which addresses this same problem.
