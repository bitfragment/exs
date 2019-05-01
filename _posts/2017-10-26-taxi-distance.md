---
title: "Taxi distance"
date: 2017-10-26
tags: python
---

## Source

Source: Advent of Code [2016](http://adventofcode.com/2016),
[Day 1: No Time for a Taxicab](http://adventofcode.com/2016/day/1)


## Description

Given an array `p` of strings representing instructions, each 
in the format `Ln`  or `Rn` where `n` is an integer,
return the [taxicab distance] or "Manhattan distance" in blocks
from point `0, 0` on a street grid to the point reached by
following the sequence of instructions.

For each instruction, turn either left (`L`) or right (`R`)
90 degrees, then walk `n` blocks forward.

```
['R2', 'L3'] ⟹ 5
['R5', 'L5', 'R5', 'R3'] ⟹ 12
['R2', 'R2', 'R2'] ⟹ 2
```

[taxicab distance]: https://en.wikipedia.org/wiki/Taxicab_geometry


## My solution

```py
nsew = {
    'north': {
        'R': 'east',
        'L': 'west', 
        'walk': lambda x, y, steps: (x, y + steps)
    },
    'south': {
        'R': 'west', 
        'L': 'east',
        'walk': lambda x, y, steps: (x, y - steps)
    },
    'east': {
        'R': 'south',
        'L': 'north',
        'walk': lambda x, y, steps: (x + steps, y)
    },
    'west': {
        'R': 'north',
        'L': 'south',
        'walk': lambda x, y, steps: (x - steps, y)
    }
}

        
def taxidistance(path, x=0, y=0, direc='north'):
    x0, y0 = x, y
    for turn in path:
        rl, steps = turn[0], int(turn[1])
        direc = nsew[direc][rl]           # update direction faced
        walk = nsew[direc]['walk']        # get walk function
        x, y = walk(x, y, steps)          # walk in current direction
    return abs(x0 - x) + abs(y0 - y)      # via formula
```


## Tests

```py
import pytest
from taxidistance import *


def test_taxidistance():
    """Should calculate taxi distance correctly."""
    assert taxidistance(['R2', 'L3']) == 5
    assert taxidistance(['R5', 'L5', 'R5', 'R3']) == 12
    assert taxidistance(['R2', 'R2', 'R2']) == 2
```
