---
title: "Reverse array"
date: 2017-11-20
tags: snobol
---

## Source

HackerRank > Data Structures > Arrays > [Arrays - DS]

[Arrays - DS]: https://www.hackerrank.com/challenges/arrays-ds


## Description

Print elements of an array in reverse order

Given an array of numeric, character, or string values,
print each element in reverse order as a single line of
space-separated elements.

```
[1, 4, 3, 2]         ⟹ "2 3 4 1"
["a", "b", "c", "d"] ⟹ "d c b a"
```


## My solution

Written in Snobol4 and compiled with The Macro Implementation of
SNOBOL4 in C (CSNOBOL4) Version 1.5 by Philip L. Budne,
October 1, 2013. SNOBOL4 (Version 3.11, May 19, 1975),
Bell Telephone Laboratories.

```snobol
    i = 5
    arr = array(i)
    arr<1> = 'A♦'
    arr<2> = 'K♦'
    arr<3> = 'Q♦'
    arr<4> = 'J♦'
    arr<5> = '10♦'

lp  ge(i, 1)          :f(end)
    output = arr<i>
    i = i - 1         :(lp)

end
```


## Output

```shell
10♦
J♦
Q♦
K♦
A♦
```
