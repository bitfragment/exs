---
title: "Reverse string"
date: 2017-11-22
tags: snobol
---

## Description

Given a string, return it reversed.

```
"foobar" ⟹ "raboof"
```

Written in Snobol4 and compiled with The Macro Implementation of
SNOBOL4 in C (CSNOBOL4) Version 1.5 by Philip L. Budne,
October 1, 2013. SNOBOL4 (Version 3.11, May 19, 1975),
Bell Telephone Laboratories.


## My solution

```snobol
f1        define('reverse(s)','lp')   :(test)
lp        s arb len(1) . ch = ''      :f(reverse)
          rstr = ch rstr              :(lp)
reverse   reverse = rstr              :(return)

test      res = reverse('foobar')
          ident(res, 'raboof')        :s(end)

          output = 'Error, expected "raboof", actual: ' res

end
```

