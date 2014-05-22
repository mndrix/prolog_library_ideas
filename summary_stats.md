# Summary Statistics

Create a library with efficient implementations for many summary statistics.  I've written each of these more times than I care to remember.  Possible statistics are:

  * mean
  * median (2-quantiles)
  * deciles (10-quantiles)
  * standard deviation
  * variance
  * [trimean](http://en.wikipedia.org/wiki/Trimean)
  
Ideally, each implementation would be require only a single pass through the data.  For example, [such algorithms exist for standard deviation](http://en.wikipedia.org/wiki/Standard_deviation#Rapid_calculation_methods).
