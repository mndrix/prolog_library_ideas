# YAML Library

Parse and generate YAML text.  Lists and scalars can be represented in Prolog natively.  Maps are more difficult.  I could represent them as lists of pairs or using `library(assoc)` or the speculated [library(map)](map.md).  Either way, this would be a fantastic place to use quasi quotation since one could do this:

```prolog
calculate_age(today, Birth, Age),
YAML = <![yaml[
    name: Michael Hendricks
    age: $Age
    hobbies:
        - programming
        - hiking
        - reading
]]>,
save_config_file(‘config.yaml’, YAML).
```

It’s probably best to write a generic parser and generator with DCG.  Then implement a quasi quotation library on top of that.
