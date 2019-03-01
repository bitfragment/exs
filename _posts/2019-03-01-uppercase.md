---
title: "To uppercase"
date: 2019-03-01
tags: fortran
---

Transform a string to uppercase. With optional parameters
`begin` and `end`, capitalize the substring between
`begin` and `end`. Otherwise, capitalize the entire string.

```
"foo" ⟹ "FOO"
"foobar" 1 3 ⟹  "fOObar"
```

Compiled as Fortran 2008 by `gfortran`.

```fortran
function toupper(chr)
    character :: chr, toupper
    integer   :: idx
    character(26), parameter :: UPPER_ASC_ENG = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    character(26), parameter :: LOWER_ASC_ENG = 'abcdefghijklmnopqrstuvwxyz'

    ! Don't transform any character that is already lowercase.
    if (scan(LOWER_ASC_ENG, chr) == 0) then
        toupper = chr
    else
        idx = index(LOWER_ASC_ENG, chr)
        toupper = UPPER_ASC_ENG(idx:idx)
    end if
end


function toupper_s(str)
    character(*) :: str
    character(len(str)) :: buf, toupper_s
    integer :: i
    do i = 1, len(str)
        buf(i:i) = toupper(str(i:i))
    end do
    toupper_s = buf
end


function toupper_subs(str, begin, end)
    character(*) :: str
    character(len(str)) :: buf, toupper_subs
    integer, optional :: begin, end
    integer :: b, e, i
    buf = str; b = 1; e = len(str)
    if (present(begin)) b = begin
    if (present(end)) e = end
    do i = b, e
        buf(i:i) = toupper(str(i:i))
    end do
    toupper_subs = buf
end
```
