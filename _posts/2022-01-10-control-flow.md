---
title: 'Control flow'
date: 2022-01-10
tags: forth
src-author: Leo Brodie
src-title: '<em>Starting Forth</em>, Chapter 4: Decisions, Decisions…'
src-url: https://www.forth.com/starting-forth/4-conditional-if-then-statements/
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Problem 1

> What will the phrase `0= 0=` leave on the stack when the argument is

1. -1
2. 0
3. 200

My solution:

```forth
: p1 ( n -- n ) 0= 0= ;
: test-p1 ( -- ) 
     -1 p1 assert( 0< )
      0 p1 assert( 0= ) 
    200 p1 assert( 0< )
;
test-p1
```

## Problem 3

> Define a word called CARD which, given a person’s age on the stack,
> prints out either of these two messages (depending on the relevant
> laws in your area): `ALCOHOLIC BEVERAGES PERMITTED` or `UNDER AGE`

My solution, designed to be testable by separating result and message:

```forth
: p3 ( n -- n ) 21 >= ;
: p3-msg ( n -- ) p3 if ." OF AGE " else ." UNDER AGE " then ;
: test-p3 ( -- )
    20 p3 assert( 0= )   20 p3-msg ." / "
    21 p3 assert( 0< )   21 p3-msg ." / "
    22 p3 assert( 0< )   22 p3-msg
;
test-p3
```

## Problem 4

> Define a word called `SIGN.TEST` that will test a number on the stack
> and print out one of three messages: `POSITIVE` or `ZERO` or
> `NEGATIVE`

My first solution, maximally explicit:

```forth
: p4 ( n -- )
    dup 0= if ." ZERO " else
    dup 0< if ." NEGATIVE " else
              ." POSITIVE "
    then then drop
;
```

My second solution, using the logical result of `IF`.

- Duplicate the original value on the stack, then evaluate it with `if`
  (which removes the duplicated value).
- If the original value is zero, execution skips to `else` on line 3. We
  then remove the original value placed on the stack.
- If the top value is nonzero, it is duplicated, then compared with 0.
  This procedure removes the duplicated value and adds the logical flag
  designating the result of the comparison with 0. In examining that
  result, `if` removes that flag. The original value placed on the stack
  is then removed.

```forth
: p4 ( n -- )
    dup if
        dup 0 < if ." NEGATIVE " else ." POSITIVE " then
    else ." ZERO " then drop
;
```

Debug:

```forth
0 dbg p4 
: p4  
Scanning code...

Nesting debugger ready!
[ 1 ] 00000 
10FE92560 10FDAB838 dup            -> [ 2 ] 00000 00000 
10FE92568 10FDAB458 IF             -> [ 1 ] 00000 
10FE92668 10FDAB450 .\" ZERO "     -> ZERO [ 1 ] 00000 
10FE926B0 10FDAB828 THEN drop      -> [ 0 ] 
10FE926B8 10FDAB428 ;              ->  ok
.s <0>  ok

clearstack -1 dbg p4 
: p4  
Scanning code...

Nesting debugger ready!
[ 1 ] 18446744073709551615 
10FE92560 10FDAB838 dup            -> [ 2 ] 18446744073709551615 18446744073709551615 
10FE92568 10FDAB458 IF             -> [ 1 ] 18446744073709551615 
10FE92578 10FDAB838 dup            -> [ 2 ] 18446744073709551615 18446744073709551615 
10FE92580 10FDAB560 0              -> [ 3 ] 18446744073709551615 18446744073709551615 00000 
10FE92590 10FDAB6C0 <              -> [ 2 ] 18446744073709551615 18446744073709551615 
10FE92598 10FDAB458 IF             -> [ 1 ] 18446744073709551615 
10FE925A8 10FDAB450 .\" NEGATIVE " -> NEGATIVE [ 1 ] 18446744073709551615 
10FE925F8 10FDAB450 ELSE           -> [ 1 ] 18446744073709551615 
10FE92658 10FDAB450 THEN ELSE      -> [ 1 ] 18446744073709551615 
10FE926B0 10FDAB828 THEN drop      -> [ 0 ] 
10FE926B8 10FDAB428 ;              ->  ok
.s <0>  ok
```

My third solution, designed to be testable by leaving symbolic values on
the stack: -1 for negative, else 1 for positive, else 0.

```forth
: p4 ( n -- n )
    dup if
        dup 0 < if drop -1 else drop 1 then
    else drop 0 then
;

: test-p4 ( -- )
     0   p4 assert( 0= )
    -1   p4 assert( 0< )
    -123 p4 assert( 0< )
     1   p4 assert( 0> )
     123 p4 assert( 0> )
;

test-p4
```

## Execute this file

```txt
```
