---
title: 'Postfix arithmetic'
date: 2021-12-10
tags: forth
src-author: Leo Brodie
src-title: '<em>Starting Forth</em>, Chapter 2: How to Get Results'
src-url: https://www.forth.com/starting-forth/2-stack-manipulation-operators-arithmetic/
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Postfix Practice Problems

> Convert the following infix equations to postfix "calculator style."
> 1. c(a+b)
> 2. (3a - b) / 4 + c
> 3. (0.5 ab) / 100
> 4. (n + 1) / n
> 5. x(7x + 5)

### My solution

1. a b + c * ✅
2. 3 a * b - 4 c + / ❌ should be 3 a * b - 4 / c +
3. 0.5 a * b * 100 / ❌ should be a b * 100 / 2 / or a variation
4. n 1 + n / ✅
5. 7 x * 5 + x * ✅

> Convert the following postfix expressions to infix:
> 1. a b - b a + /
> 2. a b 10 * /

1. (a - b) / (b + a) ✅
2. a / b10 ✅

## Definition Style Practice Problems

> Convert the following infix expressions into Forth definitions and show 
> the stack order required by your definitions.
> 1. (a - 4b) / 6 + c
> 2. a / (8b)
> 3. 0.5 ab / 100
> 4. a(2a + 3)
> 5. (a - b) / c

### My solution

Problem #5 can't be solved without stack manipulation operators not
yet introduced.

```forth
: pp1 ( c a b -- result ) 4 * - 6 / + ;
: test-pp1 assert( 3 1 2 pp1 1 = ) ;
test-pp1

: pp2 ( a b -- result ) 8 * / ;
: test-pp2 assert( 1 2 pp2 0 = ) ;
test-pp2

: pp3 ( a b -- result ) * 100 * 2 / ;
: test-pp3 assert( 1 2 pp3 100 = ) ;
test-pp3

: pp4 ( a a -- result ) 2 * 3 + * ;
: test-pp4 assert( 2 2 pp4 14 = ) ;
test-pp4
```

## Execute this file

```txt
$ codedown forth < 2021-12-10-postfix-arithmetic.md | grep . | gforth
Gforth 0.7.3, Copyright (C) 1995-2008 Free Software Foundation, Inc.
Gforth comes with ABSOLUTELY NO WARRANTY; for details type `license'
Type `bye' to exit
: pp1 ( c a b -- result ) 4 * - 6 / + ;  ok
: test-pp1 assert( 3 1 2 pp1 1 = ) ;  ok
test-pp1  ok
: pp2 ( a b -- result ) 8 * / ;  ok
: test-pp2 assert( 1 2 pp2 0 = ) ;  ok
test-pp2  ok
: pp3 ( a b -- result ) * 100 * 2 / ;  ok
: test-pp3 assert( 1 2 pp3 100 = ) ;  ok
test-pp3  ok
: pp4 ( a a -- result ) 2 * 3 + * ;  ok
: test-pp4 assert( 2 2 pp4 14 = ) ;  ok
test-pp4  ok
```