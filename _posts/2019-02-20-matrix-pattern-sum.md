---
title: "Matrix pattern sum"
date: 2019-02-20
tags: python
---


## Source

HackerRank: [2D Array - DS]

[2D Array - DS]: https://www.hackerrank.com/challenges/2d-array/problem

Given a six integer value by six integer value matrix, return the
maximum of the sums derived from each of the sixteen "hourglass"
patterns it contains.

Hourglass pattern:

```
x x x
  x
x x x
```

```py
>>> m = [\
[1, 1, 1, 0, 0, 0],\
[0, 1, 0, 0, 0, 0],\
[1, 1, 1, 0, 0, 0],\
[0, 0, 2, 4, 4, 0],\
[0, 0, 0, 2, 0, 0],\
[0, 0, 1, 2, 4, 0] \
]
>>> maxh(m)
19
```


## My solution

```py
def sumh(mtrx: List, row: int, col: int) -> int:
    '''Return the sum of the hourglass pattern.

    The pattern is in a 3 x 3 matrix at `row`, `col`.

    >>> m = [\
    [1, 1, 1, 0, 0, 0],\
    [0, 1, 0, 0, 0, 0],\
    [1, 1, 1, 0, 0, 0],\
    [0, 0, 2, 4, 4, 0],\
    [0, 0, 0, 2, 0, 0],\
    [0, 0, 1, 2, 4, 0] \
    ]
    >>> sumh(m, 0, 0)
    7
    '''
    top = mtrx[row][col: col + 3]
    middle = mtrx[row + 1][col + 1]
    bottom = mtrx[row + 2][col: col + 3]
    return sum(top) + middle + sum(bottom)


def maxh(mtrx: List) -> int:
    '''Return the maximum of the hourglass-pattern sums.

    >>> m = [\
    [1, 1, 1, 0, 0, 0],\
    [0, 1, 0, 0, 0, 0],\
    [1, 1, 1, 0, 0, 0],\
    [0, 0, 2, 4, 4, 0],\
    [0, 0, 0, 2, 0, 0],\
    [0, 0, 1, 2, 4, 0] \
    ]
    >>> maxh(m)
    19
    '''
    return max(
        sumh(mtrx, row, col)
        for row in range(4)
        for col in range(4)
        )
```


## Tests

```py
test_data = {
    'm1':    [[1, 1, 1, 0, 0, 0],
              [0, 1, 0, 0, 0, 0],
              [1, 1, 1, 0, 0, 0],
              [0, 0, 0, 0, 0, 0],
              [0, 0, 0, 0, 0, 0],
              [0, 0, 0, 0, 0, 0]],

    'm2':    [[1, 1, 1, 0, 0, 0],
              [0, 1, 0, 0, 0, 0],
              [1, 1, 1, 0, 0, 0],
              [0, 0, 2, 4, 4, 0],
              [0, 0, 0, 2, 0, 0],
              [0, 0, 1, 2, 4, 0]],

    'm3':    [[-9, -9, -9,  1, 1, 1],
              [ 0, -9,  0,  4, 3, 2],
              [-9, -9, -9,  1, 2, 3],
              [ 0,  0,  8,  6, 6, 0],
              [ 0,  0,  0, -2, 0, 0],
              [ 0,  0,  1,  2, 4, 0]]
}


def test_mytest_sumh():
    m = test_data['m1']
    assert sumh(m, 0, 0) == 7


def test_mytest_maxh():
    m = test_data['m1']
    assert maxh(m) == 7


def test_provided_maxh():
    m = test_data['m2']
    assert maxh(m) == 19
    m = test_data['m3']
    assert maxh(m) == 28
```


## Extras

```py
def interactive(solution):
    lines = stdin.readlines()
    mtrx = [[int(x) for x in line.split(' ')] for line in lines]
    print(solution(mtrx))


def tooling():
    from os import system
    from sys import stdin
    from typing import List
    import doctest
    import pytest
    system(f'mypy {__file__} --ignore-missing-imports')
    system(f'flake8 {__file__}')
    doctest_ret = doctest.testmod()
    if doctest_ret[0] == 0:
        pytest_flags = ['-q', '-x', '--cov=./']
        pytest_flags.append(__file__)
        pytest_ret = pytest.main(pytest_flags)
        if pytest_ret == 0:
            system('coverage html')


if __name__ == '__main__':
    from sys import argv
    try:
        if argv[1] == "-i":
            interactive(maxh)
    except IndexError:
        tooling()
```
