---
title: "List operations"
date: 2019-02-22
tags: python
---

Source: Werner Hett, [Prolog Problems]

[Prolog Problems]: https://sites.google.com/site/prologsite/prolog-problems

## 1.01 Find the last element of a list
```py
def my_last(xs):
    return xs[-1]


assert my_last((1, 2, 3)) == (1, 2, 3)[2]
```

## 1.02 Find the last but one (second to last) element of a list
```py
def last_but_one(xs):
    return xs[-2]


assert last_but_one((1, 2, 3)) == (1, 2, 3)[1]
```

## 1.03 Find the K'th element of a list
```py
def element_at(xs, k: int):
    return xs[k - 1]


assert element_at((1, 2, 3), 2) == 2
```

## 1.04 Find the number of elements of a list
```py
def num_elts(xs):
    return len(xs)


assert num_elts((1, 2, 3)) == 3
```

## 1.05 Reverse a list
```py
def my_reverse(xs):
    return list(reversed(xs))


assert my_reverse([1, 2, 3]) == [3, 2, 1]
```

## 1.06 Find out whether a list is a palindrome
```py
def is_palindrome(xs):
    return list(reversed(xs)) == xs


assert is_palindrome([1, 2, 3, 2, 1])
assert not is_palindrome([1, 2, 3, 4, 5])
```

## 1.07 Flatten a nested list structure
```py
def my_flatten(xs):
    """Handles only shallow nesting before exceeding recursion depth."""
    ret = []
    for x in xs:
        if isinstance(x, list):
            ret.extend(my_flatten(x))
        else:
            ret.append(x)
    return ret


assert my_flatten([1, [2, [3, 4], 5]]) == [1, 2, 3, 4, 5]
```

## 1.08 Eliminate consecutive duplicates of list elements
```py
def compress(xs):
    ret = [xs[0]]
    for elt in xs[1:]:
        if elt == ret[-1]:
            continue
        else:
            ret.append(elt)
    return ret


assert compress([1, 1, 1, 1, 2, 3, 3, 1, 1, 4, 5, 5, 5, 5]) == [1, 2, 3, 1, 4, 5]
```

## 1.09 Pack consecutive duplicates of list elements into sublists
```py
def pack(xs):
    ret = [[xs[0]]]
    for elt in xs[1:]:
        if elt == ret[-1][-1]:
            ret[-1].append(elt)
        else:
            ret.append([elt])
    return ret


assert pack([1, 1, 1, 1, 2, 3, 3, 1, 1, 4, 5, 5, 5, 5]) == [
    [1, 1, 1, 1],
    [2],
    [3, 3],
    [1, 1],
    [4],
    [5, 5, 5, 5],
]
```

## 1.10 Run-length encoding of a list
```py
def encode(xs):
    packed = pack(xs)
    return [[len(x), x[0]] for x in packed]


assert encode(
    ["a", "a", "a", "a", "b", "c", "c", "a", "a", "d", "e", "e", "e", "e"]
) == [[4, "a"], [1, "b"], [2, "c"], [2, "a"], [1, "d"], [4, "e"]]
```
