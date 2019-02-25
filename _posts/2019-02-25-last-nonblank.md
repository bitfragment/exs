---
title: "Last nonblank character"
date: 2019-02-25
tags: fortran
---

Return the last nonblank character in a string.

```
"foo"  ⟹ "o"
"foo " ⟹ "o"
```

Compiled as Fortran 2008 by `gfortran.`

```fortran
function last_nonblank(str)
    character(*) :: str
    character    :: last_nonblank
    integer      :: strlen
    strlen = len_trim(str)
    last_nonblank = str(strlen:strlen)
end
```
