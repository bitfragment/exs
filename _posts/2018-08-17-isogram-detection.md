---
title: "Isogram detection"
date: 2018-08-17
tags: java
---

Source: [Cracking the Coding Interview](http://www.crackingthecodinginterview.com/)

* TOC
{:toc}

Description
-----------
Determine if a string is an isogram (is composed of unique characters)

```
> aa
false
> ab
true
```


Notes
-----
* ASCII and Unicode represent different constraints (or assumptions)
  here.
* Of the provided solutions, the best uses a bit vector to optimize for
  space complexity.
* Other possible, but probably not comparably optimal solutions include:
  - Comparing each character in the string to every other character 
    in the string, which is O(n^2) time and O(1) space
  - Sorting the string in place, if it could be done in O(n log n), 
    and check the string for contiguous identical characters. Space 
    complexity is an open question here.


My solution
-----------
A brute force solution: store each character in an array
after checking it, and check the array of checked characters
at each step, returning as soon as a duplicate character is
detected. O(n) where *n* is the length of the string, because
in the worst case it checks every character.

```java
public static boolean isUnique(String s) {
    int slen = s.length();
    if (slen == 0) return false;
    if (slen == 1) return true;
    char[] checked = new char[slen];
    for (int i = 0; i < slen; i++) {
        char thisChar = s.charAt(i);
        if (indexOf(checked, slen, thisChar) >= 0) {
            return false;
        } else {
            checked[i] = thisChar;
        }
    }
    return true;
}

public static int indexOf(char[] arr, int arrlen, char c) {
    for (int i = 0; i < arrlen; i++) {
        if (arr[i] == c) return i;
    }
    return -1;
}
```


Provided solutions
------------------

### 1. Use an array of boolean values

Also O(n) where *n* is the length of the string, but the
logic here is quite different, and more apposite than that
of my solution. Why bother equality-checking two character 
representations when the question of whether a character has
recurred has a boolean answer?

Another way to bail early, which hadn't occurred to me, is
to use the constraint of the size of the (assumed) character
set.

```java
public static boolean isUnique(String s) {
    int slen = s.length();
    if (slen < 1 || slen > 128) return false; // Assume 128 chars
    if (slen == 1) return true;

    boolean[] checked = new boolean[128];
    for (int i = 0; i < slen; i++) {
        char thisChar = s.charAt(i);
        if (checked[thisChar]) return false;
        checked[thisChar] = true;
    }
    return true;
}
```

### 2. Use a bit vector

Space complexity of the previous solution is O(1). It can be 
reduced by a factor of eight this way. The following assumes
that a string uses only lowercase a-z, so we can use a single
`int` (32 bits) as the bit vector.

At this point, my level of understanding of bitwise operations
is enough to explain what we're doing here, but not to explain
why it works.

We start with a "checker" value of 0, then, for each character
in the input string, we do the following.
1. Derive an integer value between 0-26 by subtracting `a`
2. Left-shift that value
3. Use bitwise AND to determine if this character has been seen
   before (and return false if so)
4. Set the checker value to the result of bitwise OR with the
   shifted value

```java
public static boolean isUnique(String s) {
    int slen = s.length();
    if (slen < 1 || slen > 26) return false; // Assume 26 chars
    if (slen == 1) return true;

    int checker = 0;
    for (int i = 0; i < slen; i++) {
        int val = s.charAt(i) - 'a';
        int shifted = 1 << val;
        if ((checker & shifted) > 0) return false;
        checker |= shifted;
    }
    return true;
}
```


Tests
-----

```java
@Test
public void testIsUnique() {
    assertFalse(isUnique(""));
    assertTrue(isUnique("a"));
    assertFalse(isUnique("aa"));
    assertTrue(isUnique("ab"));
    assertFalse(isUnique("aba"));
    assertTrue(isUnique("abc"));
}

@Test
public void testIndexOf() {
    char[] arr = {'a', 'b', 'c'};
    assertEquals(indexOf(arr, 3, 'b'), 1);
    assertEquals(indexOf(arr, 3, 'd'), -1);
}
```
