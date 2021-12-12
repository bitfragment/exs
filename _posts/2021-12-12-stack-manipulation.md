---
title: 'Stack manipulation'
date: 2021-12-11
tags: forth
src-author: Leo Brodie
src-title: '<em>Starting Forth</em>, Chapter 2: How to Get Results'
src-url: https://www.forth.com/starting-forth/2-stack-manipulation-operators-arithmetic/
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Stack Manipulation and Math Definitions

### Problem 1

> Write a phrase that flips three items on the stack, leaving the middle
> number in the middle; that is, `a b c` becomes `c b a`.

My solution:

```forth
: rev3 ( n1 n2 n3 -- n3 n2 n1 ) swap rot ;
( 1 2 3 dbg rev3 )
: test-rev3 1 2 3 rev3 assert( 1 = . + 5 = ) ;
test-rev3
```

### Problem 2

> Write a phrase that does what `OVER` does, without using `OVER`.

My solution:

```forth
: over ( n1 n2 -- n1 n2 n1 ) swap dup rot swap ;
( 1 2 dbg over )
: test-over 1 2 over assert( + 3 = 1 ) ;
test-over
```

### Problem 3

> Write a definition called `-ROT`, which rotates the top three stack
> items in the opposite direction from ROT; that is, `a b c` becomes `c
> a b`.

My solution:

```forth
: -rot ( n1 n2 n3 -- n3 n1 n2 ) rot rot ;
( 1 2 3 dbg -rot )
: test--rot 1 2 3 -rot assert( + = 3 = 3 ) ;
test--rot
```

### Problem 4

> Write definitions for the following equations, given the stack effects
> shown:

My solutions:

`(n+1) / n ( n -- result )`

```forth
: equation1 ( n -- result ) dup 1 + swap / ;
( 1 dbg equation1 )
: test-equation1 1 equation1 assert( = 2 ) ;
test-equation1
```

`x(7x + 5) ( x -- result )`

```forth
: equation2 ( x -- result ) dup 7 * 5 + * ;
( 2 dbg equation2 )
: test-equation2 2 equation2 assert( 38 = ) ;
test-equation2
```

`9a^2 - ba ( a b -- result )`

```forth
: equation3 ( a b -- result ) swap dup dup * 9 * rot rot * - ;
( 2 3 dbg equation3 )
: test-equation3 2 3 equation3 assert( = 30 ) ;
test-equation3
```

A better solution given in the book: `over 9 *  swap - *`

## Problems — Chapter 2

### Problem 1

> What’s the difference between DUP DUP and 2DUP? 

My solution:

`2dup` duplicates the first pair of items on the stack:

```forth
clearstack 1 2 2dup + + + 6 = 0= s" test failed " exception and throw
```

Whereas `dup dup` duplicates the first item on the stack, twice:

```forth
clearstack 1 2 dup dup + + + 7 = 0= s" test failed " exception and throw
```

### Problem 2

> Write a phrase which will reverse the order of the top four items on
> the stack; that is, `( 1 2 3 4 -- 4 3 2 1 )`.

My solution:

```forth
: 4rev ( n1 n2 n3 n4 -- n4 n3 n2 n1 ) swap rot swap rot ;
( 1 2 3 4 dbg 4rev )
: test-4rev ( -- f ) 1 2 3 4 4rev assert( + 7 = . + 3 = ) ;
test-4rev
```

Solution given in the book uses `swap 2swap swap`.

### Problem 3

> Write a definition called 3DUP which will duplicate the top three
> numbers on the stack; for example, `( 1 2 3 -- 1 2 3 1 2 3 )`.

My solution:

```forth
: 3dup ( n1 n2 n3 -- n1 n2 n3 n1 n2 n3 ) dup 2over rot ;
( 1 2 3 dbg 3dup )
: test-3dup 1 2 3 3dup assert( 3 = . 2 = . 1 = . 3 = . 2 = . 1 = ) ;
test-3dup
```

### Problem 5

> Write a set of words to compute prison sentences for hardened
> criminals such that the judge can enter: `CONVICTED-OF ARSON HOMICIDE
> TAX-EVASION↵ok WILL-SERVE↵35 years ok` or any series of crime
> beginning with the word `CONVICTED-OF` and ending with `WILL-SERVE`.
> Use these sentences: `HOMICIDE 20 years ARSON 10 years BOOKMAKING 2
> years TAX-EVASION 5 years`.

My solution:

Computers should not be used for this purpose.

Different problem, similar logic:

```forth
: added ( -- 0 ) 0 ;

: meat ( total -- total+10 ) 10 + ;
: fish ( total -- total+10 ) 10 + ;
: fruit ( total -- total+8 ) 8 + ;
: vegetables ( total -- total+8 ) 8 + ;
: chocolate ( total -- total+5 ) 5 + ;
: wine ( total -- total+9 ) 9 + ;

: basket ( total -- ) . ." usd " ;

: test-added added meat fish fruit assert( dup = 28 ) ;
test-added

added meat fish fruit basket
```

### Problem 6

> You’re the inventory programmer at Maria’s Egg Ranch. Define a word
> called `EGG.CARTONS` which expects on the stack the total number of
> eggs laid by the chickens today and prints out the number of cartons
> that can be filled with a dozen each, as well as the number of
> leftover eggs.

```forth
: egg.cartons ( total -- ) 12 /mod . ." cartons " . ." egg(s) left over " ;
1202 egg.cartons
```

## Execute this file

```txt
$ codedown forth < 2021-12-12-stack-manipulation.md | grep . | gforth
Gforth 0.7.3, Copyright (C) 1995-2008 Free Software Foundation, Inc.
Gforth comes with ABSOLUTELY NO WARRANTY; for details type `license'
Type `bye' to exit
: rev3 ( n1 n2 n3 -- n3 n2 n1 ) swap rot ;  ok
( 1 2 3 dbg rev3 )  ok
: test-rev3 1 2 3 rev3 assert( 1 = . + 5 = ) ;  ok
test-rev3 -1  ok
: over ( n1 n2 -- n1 n2 n1 ) swap dup rot swap ; redefined over   ok
( 1 2 dbg over )  ok
: test-over 1 2 over assert( + 3 = 1 ) ;  ok
test-over  ok
: -rot ( n1 n2 n3 -- n3 n1 n2 ) rot rot ; redefined -rot   ok
( 1 2 3 dbg -rot )  ok
: test--rot 1 2 3 -rot assert( + = 3 = 3 ) ;  ok
test--rot  ok
: equation1 ( n -- result ) dup 1 + swap / ;  ok
( 1 dbg equation1 )  ok
: test-equation1 1 equation1 assert( = 2 ) ;  ok
test-equation1  ok
: equation2 ( x -- result ) dup 7 * 5 + * ;  ok
( 2 dbg equation2 )  ok
: test-equation2 2 equation2 assert( 38 = ) ;  ok
test-equation2  ok
: equation3 ( a b -- result ) swap dup dup * 9 * rot rot * - ;  ok
( 2 3 dbg equation3 )  ok
: test-equation3 2 3 equation3 assert( = 30 ) ;  ok
test-equation3  ok
clearstack 1 2 2dup + + + 6 = 0= s" test failed " exception and throw  ok
clearstack 1 2 dup dup + + + 7 = 0= s" test failed " exception and throw  ok
: 4rev ( n1 n2 n3 n4 -- n4 n3 n2 n1 ) swap rot swap rot ;  ok
( 1 2 3 4 dbg 4rev )  ok
: test-4rev ( -- f ) 1 2 3 4 4rev assert( + 7 = . + 3 = ) ;  ok
test-4rev -1  ok
: 3dup ( n1 n2 n3 -- n1 n2 n3 n1 n2 n3 ) dup 2over rot ;  ok
( 1 2 3 dbg 3dup )  ok
: test-3dup 1 2 3 3dup assert( 3 = . 2 = . 1 = . 3 = . 2 = . 1 = ) ;  ok
test-3dup -1 -1 -1 -1 -1  ok
: added ( -- 0 ) 0 ;  ok
: meat ( total -- total+10 ) 10 + ;  ok
: fish ( total -- total+10 ) 10 + ;  ok
: fruit ( total -- total+8 ) 8 + ;  ok
: vegetables ( total -- total+8 ) 8 + ;  ok
: chocolate ( total -- total+5 ) 5 + ;  ok
: wine ( total -- total+9 ) 9 + ;  ok
: basket ( total -- ) . ." usd " ;  ok
: test-added added meat fish fruit assert( dup = 28 ) ;  ok
test-added  ok
added meat fish fruit basket 28 usd  ok
: egg.cartons ( total -- ) 12 /mod . ." cartons " . ." egg(s) left over " ;  ok
1202 egg.cartons 100 cartons 2 egg(s) left over  ok
```
