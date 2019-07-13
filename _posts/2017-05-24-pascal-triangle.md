---
title: "Pascal's triangle"
date: 2017-05-24
tags: scala
---

Return number at `col`, `row` of Pascal's triangle.

```scala
scala> for (row <- 0 to 4) {
     |   for (col <- 0 to row) {
     |     print(pascal(col, row) + " ")
     |   }
     |   println()
     | }
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
```

---

```scala
def pascal(col: Int, row: Int): Int = (col, row) match {
    case (0, row) => 1
    case (col, row) if col == row => 1
    case _ => pascal(col - 1, row - 1) + pascal(col, row - 1)
}
```
