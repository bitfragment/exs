---
title: Dictionary lookup
date: 2019-06-11
tags: python
src-author: "Dan Bader"
src-title: "Python Tricks: The Book"
src-url: https://realpython.com/products/python-tricks-book/
---

**§7.1 Dictionary Default Values**

```py
mydict = {
    'a': 1,
    'b': 2,
    'c': 3,
}
```

One way to avoid a `KeyError`:

```py
try:
    x = mydict['d']
except KeyError:
    x = 0
    
assert x == 0
```

Another: use `dict.get()`, which returns `None`:

```py
y = mydict.get('d')
assert y is None
```

With the optional second argument representing a default value, returns that:

```py
z = mydict.get('e', 0)
assert z == 0
```
