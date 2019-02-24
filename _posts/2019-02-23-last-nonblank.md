---
title: "Last nonblank character"
date: 2019-02-23
tags: cobol
---

Return the last nonblank character in a string.

```
"foo"  ⟹ "o"
"foo " ⟹ "o"
```
Compiled as Cobol 2002 by `cobc` (GNU Cobol) 1.1.0.

```
program-id. last_nonblank is initial.
    data division.
        working-storage section.
            01 trailing_spaces pic 99 value zeros.
            01 len pic 99 value zeros.
        linkage section.
            01 str pic x(80).
            01 chr pic x.
    procedure division using str, chr.
        inspect function reverse(str) tallying trailing_spaces for leading spaces.
        compute len = function length(str) - trailing_spaces.
        move str(len:) to chr.
end program last_nonblank.
```
