---
title: Factorial
date: 2019-05-13
src-title: Code Chef FCTRL2 Small factorials
src-url: https://www.codechef.com/problems/FCTRL2
tags: python
---

Calculate factorial of a small positive integer *n* where
1 <= *n* <= 100.

```py
def fact(n: int) -> int:
    result = 1
    if n == 1:
        return result
    for i in range(2, n + 1):
        result *= i
    return result
    

def olj():
    try:
        for _ in range(int(input())):
            n = int(input())
            print(fact(n))
    except EOFError:
        return


olj()
```
