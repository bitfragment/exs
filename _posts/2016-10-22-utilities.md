---
title: Utilities
date: 2016-10-22
tags: c
---

## Contents
{:.no_toc}

* TOC
{:toc}

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


const char VOWEL_ASC_ENG[] = "aeiouAEIOU";
```

## edgeval()

Return edge value of array. Returns highest value in array
if `high_low` is set to `1`, lowest otherwise.

```c
> long a[] = {9, 8, 7, 1, 2, 3, 11, 29, -4}
> edgeval(a, 9, 0)
-4
> edgeval(a, 9, 1)
29
```

---

```c
long edgeval(long *ns, size_t nslen, _Bool high_low) {
    long edge = ns[--nslen];
    while (nslen--) {
        if (high_low) {
            if (ns[nslen] > edge) edge = ns[nslen];
        } else {
            if (ns[nslen] < edge) edge = ns[nslen];
        }
    }
    return edge;
}
```


## is_vowel_asc_eng()

Return true if input is contained in `VOWEL_ASC_ENG`.

```c
> is_vowel_asc_eng("a")
1
> is_vowel_asc_eng("b")
0
```

---

```c
_Bool is_vowel_asc_eng(char *c) {
    return c[0] != '\0' && strstr(VOWEL_ASC_ENG, c) != NULL;
}
```

## last_nonblank()

Return last nonblank character of a string.

```c
> last_nonblank("foobar"))
'r'
> last_nonblank("foobar "))
'r'
```

---

```c
char last_nonblank(char *s) {
    int len = strlen(s) - 1;
    while (isspace(s[len])) len--;
    return s[len];
}
```

## rotate()

Rotate string `s` by `n` places (in place transformation).

```c
> char s[] = "foobar"
> rotate(s, 2)
> s
'arfoob'
```

---

```c
void rotate(char *s, int n) {
    size_t l = strlen(s);
    while (n--) {
        char last = s[l - 1];
        for (int i = l - 1; i > 0; i--) {
            s[i] = s[i - 1];
        }
        s[0] = last;
    }
}
```

## rotated()

Return copy of `s` rotated by `n` places.

```c
> rotated("foobar", 2)
'arfoob'
```

---

```c
char *rotated(char *s, int n) {
    int l = strlen(s);
    char * temp = malloc(sizeof(char) * (l + 1));
    char * out = malloc(sizeof(char) * (l + 1));
    if (temp == NULL || out == NULL) {
        fprintf(stderr, "out of memory\n");
        exit(EXIT_FAILURE);
    }
    strcpy(temp, s);
    while (n--) {
        out[0] = temp[l - 1];
        for (int i = 1; i < l; i++) {
            out[i] = temp[i - 1];
        }
        strcpy(temp, out);
    }
    out[l + 1] = '\0';
    return out;
}
```

## rotations()

Return the number of rotations required to make s1 == s2.

```c
> char s1[] = "foobar", s2[] = "arfoob"
> rotations(s1, s2)
2
```

---

```c
int rotations(char *s1, char *s2) {
    if (!strcmp(s1, s2)) return 0;
    size_t l = strlen(s1);
    for (int r = 1; r < l; r++) {
        rotate(s1, 1);
        if (!strcmp(s1, s2)) return r;
    }
    return -1;
}
```

## sum()

Sum integer values in an array.

```c
> long ns[] = {1, 2, 3, 4, 5}
> sum(ns, 5)
15
```

---

```c
long sum(long *ns, size_t nslen) {
    long total = 0;
    while (nslen--) total += *(ns++);
    return total;
}
```

## suminner()

Sum integer values in an array, excluding the
lowest and highest values.

```c
> long ns[] = {3, 2, 1, 5, 4}
> suminner(ns, 5))
9
```

---

```c
long suminner(long *ns, size_t nslen) {
    if (nslen <= 1) return 0;
    long sum = 0;
    long lowest = edgeval(ns, nslen, 0);
    int highest = edgeval(ns, nslen, 1);
    while (nslen--) sum += ns[nslen];
    return sum - lowest - highest;
}
```

## toupper_s()

Convert a string to uppercase.

```c
> toupper_s("foo")
'FOO'
```

---

```c
char *toupper_s(char *s) {
    int l = strlen(s);
    char *buf = malloc(sizeof(char) * (l + 1));
    if (buf == NULL) {
        fprintf(stderr, "Out of memory\n");
        exit(EXIT_FAILURE);
    }
    for (int i = 0; i < l; i++) {
        buf[i] = toupper(s[i]);
    }
    buf[++l] = '\0';
    return buf;
}
```


## toupper_subs()

Convert a substring to uppercase.

```c
> toupper_subs("foobar", 1, 5)
'fOOBAr'
```

---

```c
char *toupper_subs(char *s, int begin, int end) {
    int l = strlen(s);
    char *buf = malloc(sizeof(char) * (l + 1));
    if (buf == NULL) {
        fprintf(stderr, "Out of memory\n");
        exit(EXIT_FAILURE);
    }
    strncpy(buf, s, begin);
    for (int i = begin; i < end; i++) {
        buf[i] = toupper(s[i]);
    }
    if (end < l) {
        strncat(buf, &s[end], l - end);
    }
    buf[++l] = '\0';
    return buf;
}
```
