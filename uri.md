# URI library

SWI Prolog comes with [library(uri)](http://www.swi-prolog.org/pldoc/doc/swi/library/uri.pl).  It has all the right pieces for a URI library, but has some weaknesses:

  * Pieces missing from a URI are represented as unbound variables.  This confounds the idea of "unknown" with "known to be absent".
  * Too much code is needed for common relations on URIs (accessing the hostname of an HTTP URI, etc).
  * No support for scheme-specific relations.  For example, `tag` and `data` URIs have specialized operations.
  * No support for adding support for more URI schemes as they come along
  * Requires a separate library for URI quasiquotations
  * Uses atoms rather than strings internally (minor)
  
I'd like a single, high level URI library that addresses all these concerns.
  

# Possible library names

The obvious name, `uri`, is already taken.  Perhaps [urial](https://en.wikipedia.org/wiki/Urial) or [uriel](https://en.wikipedia.org/wiki/Uriel) would be appropriate.
