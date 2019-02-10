---
title: "Longest word"
date: 2019-02-03
tags: python
---

## Source

[Coderbyte: Longest Word](https://www.coderbyte.com/editor/guest:Longest%20Word:Python)


## My solution

```py
import doctest
import pytest
from os import system

from string import punctuation as punct


def longest_word(mystr: str) -> str:
    '''Return the longest word in the string, ignoring punctuation
    and returning the first of multiple longest words.

    >>> longest_word('foobar baz')
    'foobar'
    '''
    if mystr == '' or ' ' not in mystr:
        return mystr
    words: list = list(x.strip(punct) for x in mystr.split(' '))
    long_len: int = max(len(x) for x in words)
    ret: list = list(x for x in words if len(x) == long_len)
    return ret[0]


def test_mytest():
    assert longest_word('') == '', 'failed on empty string'
    assert longest_word('foo') == 'foo', 'failed on single word'
    assert longest_word('foo bar') == 'foo', 'failed on words of same length'
    assert longest_word('foobar baz') == 'foobar'


def test_provided():
    assert longest_word('I love dogs') == 'love'
    assert longest_word('fun&!! time') == 'time'
    mystr = "a confusing /:sentence:/[ this is not!!!!!!!~"
    assert longest_word(mystr) == 'confusing'


if __name__ == '__main__':
    pytest.main(['-q', __file__])
    doctest.testmod(optionflags=doctest.FAIL_FAST)
    system(f'mypy {__file__} --ignore-missing-imports')
    system(f'flake8 {__file__}')
```
