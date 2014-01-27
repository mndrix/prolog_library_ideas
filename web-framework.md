# Prolog Web Framework

Writing a web server in SWI Prolog requires a lot of repetitive, low level coding.  I'd like something that's more declarative.  For now this is a collection of use cases before proposing something concrete:

  * dispatch requests to predicates based on unification
    * based on request method (`get`, `post`, `head`, etc).
    * based on path (`foo/Bar/baz`)
  * easy access to request details (path, parameters, form, url, etc)
  * unambiguous access to request details (request as list creates ambiguity between request components)
  * support middleware to factor out common layers
    * request logging
    * proxy header handling
    * memory limits
    * request time limits
  * built on a generic web server interface like
    * [PEP33](http://www.python.org/dev/peps/pep-0333)
    * [Rack](http://rack.rubyforge.org/doc/SPEC.html)
    * [JSGI](http://jackjs.org/jsgi-spec.html)
    * [PSGI](http://search.cpan.org/~miyagawa/PSGI/PSGI.pod)
