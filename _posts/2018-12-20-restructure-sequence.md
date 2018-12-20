---
title: "Restructure a sequence"
date: 2018-12-20
tags: py
---

Source: [Python Morsels](https://www.pythonmorsels.com/)

* TOC
{:toc}


## Description

Given an iterable `nums` and a "window" of `width`, return an 
iterator of tuples of the specified width.

```py
>>> numbers = [1, 2, 3, 4, 5, 6]
>>> list(window(numbers, 2))
[(1, 2), (2, 3), (3, 4), (4, 5), (5, 6)]
```

## Notes

None.


## My solution

```py
def window(nums, width):
    '''Reformat the iterable `nums` as tuples of `width` each.
    
    Given an iterable `nums` and a "window" of `width`, return an 
    iterator of tuples of the specified width.

    >>> numbers = [1, 2, 3, 4, 5, 6]
    >>> list(window(numbers, 2))
    [(1, 2), (2, 3), (3, 4), (4, 5), (5, 6)]
    '''
    nums = list(nums)
    nlen = len(nums)
    for idx in range(nlen):
        try:
            if idx < nlen - (width - 1):
                yield tuple(nums[idx:idx + width])
        except IndexError:
            pass


# Tests

def test_window_len_even():
    numbers = [1, 2, 3, 4, 5, 6]
    result = [(1, 2), (2, 3), (3, 4), (4, 5), (5, 6)]
    assert list(window(numbers, 2)) == result

def test_iterable_input():
    numbers = [1, 2, 3, 4, 5, 6]
    squares = (n ** 2 for n in numbers)
    result = [(1, 4, 9), (4, 9, 16), (9, 16, 25), (16, 25, 36)]
    assert list(window(squares, 3)) == result
        


# Interaction 

if __name__ == '__main__':
    print('My tests:')
    import doctest, pytest
    doctest.testmod()
    pytest.main([__file__])

    print('Provided tests:')
    from os import system
    system('python test_window.py')

```


## Provided solution

Uses a deque, which is interesting but seems unnecessary for
this particular problem.

```py
from collections import deque
from itertools import islice

def window(iterable, n):
    iterator = iter(iterable)
    current = deque(islice(iterator, n), maxlen=n)
    yield tuple(current)
    for item in iterator:
        current.append(item)
        yield tuple(current)
```
