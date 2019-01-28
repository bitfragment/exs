---
title: "Anagram detection"
date: 2018-08-08
tags: python
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}

## Notes

Exercise is to detect an anagram, with bonuses for ignoring spaces,
punctuation, and diacritical marks on Latin characters (but not the characters
themselves).

## My solution

Using `isalnum()` to retain alphanumeric characters and discard everything
else, and `unicodedata.normalize()` to remove diacritical marks.

```py
import unicodedata

def fmt(s):
    s = unicodedata.normalize('NFKD', s)\
       .encode('ascii', 'ignore')\
       .decode('utf-8')
    return ''.join(sorted(c.lower() for c in s if c.isalnum()))


def is_anagram(s1, s2):
    return False if len(s1) != len(s1) else fmt(s1) == fmt(s2)
```


## Provided solutions

Were mostly identical to mine, except for a useful variation using `Counter`
to tabulate characters and use the tabulation as a basis for comparison.

```py
from collections import Counter

def is_anagram(word1, word2):
    return Counter(word1) == Counter(word2)
```
