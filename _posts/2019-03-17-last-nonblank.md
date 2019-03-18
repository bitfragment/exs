---
title: "Last nonblank character"
date: 2019-03-17
tags: lisp
---

Return last nonblank character in a string.

Compiled as Common Lisp using GNU CLISP.

```
"foo"  ⟹ "o"
"foo " ⟹ "o"
```

```lisp
(defun last-nonblank (str)
  (subseq (string-right-trim '(#\Space) str)
    (- (length (string-right-trim '(#\Space) str)) 1)))
```
