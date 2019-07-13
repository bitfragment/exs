---
title: "Balanced parentheses"
date: 2017-05-25
tags: scala
---

Return true if parentheses in list `chars` are balanced.

```scala
scala> balance("(foo (bar))".toList)
res1: Boolean = true
scala> balance("(foo bar".toList)
res2: Boolean = false
```

---

```scala
def balanceLeft(chars: List[Char], left: Int): Boolean = chars match {
  case Nil => left == 0
  case _ => chars.head match {
    case ')' => left > 0 && balanceLeft(chars.tail, left - 1)
    case '(' => balanceLeft(chars.tail, left + 1)
    case _ => balanceLeft(chars.tail, left)
  }
}

def balance(chars: List[Char]): Boolean = balanceLeft(chars, 0)
```
