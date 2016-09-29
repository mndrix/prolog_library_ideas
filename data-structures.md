# Logical Data Structures

Create a library with various data structures having nice, logical implementations.  Data structures worth including might be:

  * queue
  * stack
  * dequeue
  * priority queue
  * set
  * map (probably as a tree or HAMT)

As with my [map](maps.md) idea, these should be defined in terms of generic interfaces on top of multiple (if necessary) implementations.

See _The Craft of Prolog_ by O’Keefe for an example of a queue.  The biggest trick for each structure is finding a nice, logical, reversible interface describing the true relation between the data structure, its inputs and its outputs.
