# Like Perl's WWW::Mechanize

Interacting with REST web services using `GET`, `POST`, `DELETE`, etc. is much too painful in SWI-Prolog.  One should be able to create a browser and then operate that browser by fetching pages, parsing responses, posting data, parsing forms, etc.

One of the main goals is to seamlessly handle all HTTP verbs and to handle both HTTP and HTTPS URLs without trouble.  Right now, one can’t easily use `library(http/http_client)` with HTTPS which is unfortunate.

SWI-Prolog’s current tools have a neat feature where plugins can parse the content of HTTP responses and provide structured results.  This is especially useful when receiving JSON or HTML responses.  Unfortunately, their system applies globally to all users of the library.  I often want different requests to behave differently (one returns raw HTML content while another parses it into a parse tree).  Allow one to specify the result format with the shape of the output term.  Specifying a parsed output format fails if the output can’t be parsed according to that technique.  Of course, this is done through hooks so that modules can add extra response formats.  Like this:

```prolog
get(Mech, Url, codes(Html)),
get(Mech, Url, html5(ParsedHtml)),
get(Mech, Url, json(ParsedJson)),
```

The browser object should be able to handle HTTP caching based on HTTP headers.  One should also be able to provide a predicate which determines caching policy based on a URL, the previous response headers, the last cache update time and the last cached value.

Calling `get/2` is like `get/3` with an implicit, throw-away client.

## SSL Certificates

When using `http_open/3`, I routinely encounter SSL certificate verification errors.  The web client's defaults should closely match a real world browser.  Start by looking for one of these certificate files:

  * /etc/ssl/certs/ca-certificates.crt
  * /etc/pki/tls/certs/ca-bundle.crt
  * /etc/ssl/ca-bundle.pem
  * /usr/local/etc/openssl/cert.pem

If a certificate bundle can't be found at those locations, use the list of certificates from Mozilla as [curated by cURL](http://curl.haxx.se/docs/caextract.html).

## cURL Bindings

A high-level browser module needs a good, low-level network protocol module beneath it.  [libcurl](http://curl.haxx.se/libcurl/c/) seems to be the best known to man.  Build a library with thin bindings on top of the `libcurl` API.  Build pleasant, high-level APIs on top of this low-level API.

Probably start with the [easy interface](http://curl.haxx.se/libcurl/c/libcurl-easy.html) but leave plenty of room in the API for supporting the [multi interface](http://curl.haxx.se/libcurl/c/libcurl-multi.html) down the road.

Now that I’ve figured out how to use `http_open/3`, I could probably start with that and move to `libcurl` later, if it’s justified.

## Gumbo Bindings

A high-level browser module needs a good, low-level HTML parsing module beneath it. Use [Gumbo](https://github.com/google/gumbo-parser) as the underlying implementation.  Gumbo is Google’s pure-C HTML5 parsing library with no external dependencies.  It passes HTML5 test suites and has been tested on 2.5 billion web pages from Google’s index.

## Header vs Body

HTTP responses have a relatively short header followed by a (possibly) large body.  When processing an HTTP response, the header should be available immediately.  The body should be made available as it streams from the remote server.  Conceptually, the body is a lazy stream of bytes.

For example, I might ask to download a large, often-changing file.  If the response's `ETag` header is the same as my last request, I'll want to abandon the body and disconnect from the server as soon as possible.
