---
title: "Simple boolean evaluation"
date: 2019-02-08
tags: py
---

## Source

[CodingBat Python Warmup-1 sleep_in](https://codingbat.com/prob/p173401)


## My solution

```py
import doctest
import pytest
from os import system


def sleep_in(weekday: bool, vacation: bool) -> bool:
    '''Return True if it's not a weekday or we're on vacation.

    >>> sleep_in(False, True)
    True
    '''
    return vacation or not weekday


def test_mytest():
    pass


def test_provided():
    assert sleep_in(False, False)
    assert not sleep_in(True, False)
    assert sleep_in(False, True)


if __name__ == '__main__':
    pytest.main(['-q', __file__])
    doctest.testmod()
    system(f'mypy {__file__} --ignore-missing-imports')
    system(f'flake8 {__file__}')
```
