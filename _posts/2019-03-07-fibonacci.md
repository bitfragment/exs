---
title: "Nth in Fibonacci sequence"
date: 2019-03-07
tags: python
---

Source: Rankk.org [6. The nth term]

[6. The nth term]: https://www.rankk.org/


## My solution

```py
def nth_fibo(n):
    """Return the nth term in the Fibonacci sequence.

    Consider the sequence to begin with 1, not 0."""
    if n < 1: return 0
    if n == 1 or n == 2: return 1
    prev, next = 1, 1
    c = 2
    while c <= n:
        temp = prev + next
        c += 1
        if c == n: return temp
        prev, next = next, temp


fs = [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987]

for idx, n in enumerate(fs):
    assert nth_fibo(1 + idx) == n
```
