# Cloud Storage for Terms

I want to be able to store 1st-order terms in the cloud (massive scale, shared across apps) as if I were storing them in a local Prolog module.  For example, module `foo` is defined somewhere as a cloud module.  Then

```prolog
foo:assert(things(that,matter,to,michael)),
foo:assert(things(that,matter,to,kinsey)),
foo:things(that,matter,to,Whom).

% Whom = michael ;
% Whom = kinsey .
```

Behind the scenes, each assert stores all necessary [path indexes](https://drive.google.com/file/d/0B25G2sSG-_Mvd2dETlp2VjNyTGM/view?usp=sharing) for the term in [datastore](https://developers.google.com/appengine/docs/go/datastore/reference).  The term is an entity.  Each path is stored in a list property on that entity.  For a query, we can use [datastore's merge join](http://youtu.be/AgaL6NGpkB8?t=23m) to efficiently calculate the set intersections across the necessary path indexes.  There may be a way to have it handle set unions too.  Otherwise, we could do it in memory.

# UI

The key idea is that one should interact with the cloud data as if it were stored locally in a normal Prolog module.  Caching is handled transparently (locally and in memcache).  When defining a module that operates this way, there's probably a predicate which defines the cache TTL given a term in that module.

# API

The cloud term storage should have a very simple API that's just a partially instantiated term like `foo(_,A,hi)` and responds with JSON like `[{"A":"bar"},{"A":"baz"}]`.  That way it can be used from languages other than Prolog.
