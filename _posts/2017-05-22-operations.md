---
title: Operations
date: 2017-05-22
tags: js, core
src-title: 
src-url: 
---

## myXor()

```js
const myXor = (a, b) =>
    (a && !b) || (!a && b);
```


## myCalc()

```js
> myCalc(1, '*', 0)
0
> myCalc(1, '/', 0)
null
```

---

```js
const myCalc = (n1, op, n2) => {
    const ops = {
        '+': (a, b) => a + b,
        '-': (a, b) => a - b,
        '*': (a, b) => a * b,
        '/': (a, b) => !b ? null : a / b
    };
    return !ops[op] ? null : ops[op](n1, n2);
};
```
