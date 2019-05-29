---
title: Utilities
date: 2016-09-11
tags: c++
---

## Contents
{:.no_toc}

* TOC
{:toc}

```cpp
#include <algorithm>
#include <cctype>
#include <iostream>
#include <string>

const std::string VOWEL_ASC_ENG = "aeiouAEIOU";
```

## is_vowel_asc_eng()
Return true if input is contained in `VOWEL_ASC_ENG`.

```cpp
> is_vowel_asc_eng("a")
1
> is_vowel_asc_eng("b")
0
```

---

```cpp
bool is_vowel_asc_eng(char c) {
    return VOWEL_ASC_ENG.find(c) != std::string::npos;
}
```

## last_nonblank()
Return last nonblank character of a string.

```cpp
> last_nonblank("foobar"))
'r'
> last_nonblank("foobar "))
'r'
```

---

```cpp
char last_nonblank(std::string s) {
    int len = s.length() - 1;
    while (std::isspace(s[len])) len--;
    return s[len];
}
```

## toupper_s()
Convert a string to uppercase.

```cpp
> toupper_s("foo")
'FOO'
```

---

```cpp
std::string toupper_s(std::string s) {
    std::transform(s.begin(), s.end(), s.begin(), ::toupper);
    return s;
}
```

## toupper_subs()
Convert a substring to uppercase.

```cpp
> toupper_subs("foobar", 1, 5)
'fOOBAr'
```

---

```cpp
std::string toupper_subs(std::string s, int begin = -1, int end = 0) {
    int b = begin < 0 ? 0 : begin,
        e = !end ? s.length() : end;
    std::string subs = s.substr(b, e - b);
    std::transform(subs.begin(), subs.end(), subs.begin(), ::toupper);
    s.replace(b, e - b, subs);
    return s;
}
```
