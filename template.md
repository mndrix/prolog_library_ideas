# Templating library

I want a Prolog templating library with the following features:

  * compile templates from quasiquotes into a DCG
  * loops over lists or backtracked solutions
  * helpful defaults to handle common text generation use cases
  
Most templating libraries think of a template as a way to generate text.  I believe it's more helpful to think of a template as a way to generate code which generates text.  In Prolog, code that generates text is often done the most naturally with DCGs.  So a Prolog template should be a way to easily generate DCGs.

For example, this quasiquoted template

```text
{|template(hello)||Hello {{name}}.|}
```

might generate code like

```prolog
hello(X) -->
    "Hello ",
    template:value(X, name),
    ".".

% library code
template:value(X,Field) -->
    { is_dict(X), get_dict(Field,X,Y) },
    template:textual(Y),
    !.
template:value(X,Field) -->
    call(Field,X),
    !.
template:value(X,Field) -->
    { call(Field,X,Y) },
    template:textual(Y),
    !.
    
% template:textual//1 is a multifile predicate defining a
% textual representation for atoms, numbers, strings, etc.
```

## Syntax

One option would be to adopt the syntax and keywords of an existing template language like [mustache](https://mustache.github.io/mustache.5.html) or [Go's template](https://golang.org/pkg/text/template/).  Another option would be to make a template library that's designed from the ground up with Prolog in mind.  For the first templating package, it might be best to borrow syntax and semantics from an existing package.  That would keep me focused on quasiquote and DCG implementation rather than deciding which features are useful for a templating system.
