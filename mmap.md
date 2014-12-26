# mmap

I'd love to mmap a file and have it appear in Prolog as if it were a giant list of codes.  This would make phrase_from_file more efficient since it doesn't have to seek back and forth within the file all the time.

Ideally, extracting sublists from the giant list would just give a "slice" of the underlying bytes in memory.  That probably requires C code to create a value that looks like a list to Prolog but is actually a junk of raw memory underneath.
