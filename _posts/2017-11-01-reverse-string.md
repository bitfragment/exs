---
title: "Reverse string"
date: 2017-11-01
tags: python
---

Given a string, return it reversed.

```
"foobar" ⟹ "raboof"
```

## Solution 1

```py
def myreversed(s):
    result = ''
    slen = len(s) - 1
    while slen >= 0:
        result += s[slen]
        slen -= 1
    return result
```

## Solution 2

```py
def myreversed(s):
    return s if not s else myreversed(s[1:]) + s[0]
```

## Tests

```py
import pytest
from reverse import myreversed

def test_myreversed():
    return myreversed("foobar") == "raboof"
```
