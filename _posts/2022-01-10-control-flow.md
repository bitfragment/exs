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

Solution provided in the book:

```forth
: CARD  ( age -- )
   17 > IF ." ALCOHOLIC BEVERAGES PERMITTED "  ELSE ." UNDER AGE "  THEN 
;
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

```txt
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

Solution provided in the book:

```forth
: SIGN.TEST  ( n -- )
   DUP 0< IF ." Negative " DROP  EXIT  THEN
       0> IF ." Positive "       EXIT  THEN 
   ." Zero " ;
```

Yep, I'd have written it like that if I'd known about `exit`.


## Problem 5

> In Chap. 1, we defined a word called `STARS` in such a way that it
> always prints at least one star, even if you say `0 STARS`. Using
> the word `STARS`, define a new version of `STARS` that corrects this
> problem. 

My solution, adding a test to determine if the value is greater than 0,
separating computation from output display, and leaving values on the
stack for testing:

```forth
: star ( n -- ) 42 emit ;
: stars-msg ( n -- ) 0 do star loop ;

\ Leaves two values on the stack for testing:
\ - The number of stars displayed.
\ - (Rightmost) The boolean flag that is the result of 0>, which signals
\   if any stars were displayed.
: stars ( n -- n n )
    dup 0> dup if swap dup stars-msg else swap then swap ;

: test-stars ( -- )
    \ Should not print any stars.
    -5 stars assert( false = ) assert( -5 = )
    -1 stars assert( false = ) assert( -1 = )
     0 stars assert( false = ) assert(  0 = )
    \ Should print the specified number of stars.
     1 stars assert( true  = ) assert(  1 = )
     5 stars assert( true  = ) assert(  5 = )
;

test-stars
```

Solution provided in the book:

```forth
: STAR   [CHAR] * EMIT ;
: STARS  ( #stars -- )  0 ?DO  STAR  LOOP ;
: STARS  ( n -- )  ?DUP IF  STARS  THEN ;
```


## Problem 6

> Write the definition for the word WITHIN which expects three
> arguments: `( n lo-limit hi-limit -- )` and leaves a “true” flag only
> if “n” is within the range **low-limit <= n < hi-limit**.

My solution:

```forth
\ Leave “true” on the stack if low <= n < high.
: _within ( n low high -- bool ) rot dup rot < rot rot <= and ;
: test-_within ( -- )
     0  -1  0 _within assert( false = )
     0   0  0 _within assert( false = )
     1   2  3 _within assert( false = )

    -1  -1  0 _within assert( true = )
     0  -1  1 _within assert( true = )
     0   0  1 _within assert( true = )
     2   1  3 _within assert( true = )
;
test-_within
```

Solution provided in the book:

```forth
: WITHIN  ( n lo hi+1 -- flag )  OVER -  >R  -  R> U< ;
```

Well, these return stack operators (`>r`, `r>`, `u>`) haven't been mentioned
yet in the book, so...


## Problem 7

> Here’s a number guessing game (which you may enjoy writing more than
> anyone will enjoy playing). First you secretly enter a number onto the
> stack (you can hide your number after entering it by executing the
> word `PAGE`, which clears the terminal screen). Then you ask another
> player to enter a guess followed by the word `GUESS`, as in `100
> GUESS`. The computer will either respond “TOO HIGH,” “TOO LOW,” or
> “CORRECT!” Write the definition of `GUESS`, making sure that the
> answer-number will stay on the stack through repeated guessing until
> the correct answer is guessed, after which the stack should be clear. 

My solution:

```forth
: guess ( n -- )
    2dup = if ." CORRECT " 2drop
    else 2dup <
        if ." TOO HIGH " else ." TOO LOW " then
    drop
    then
;

: test-guess
    ." should display 'TOO HIGH' → " 1 2 guess cr
    ." should display 'TOO LOW' → " 1 0 guess cr
    ." should display 'CORRECT' → " 1 01 guess cr
;
test-guess
```

Solution provided in the book:

```forth
: GUESS ( answer guess -- answer )
   2DUP = IF ." Correct! " 2DROP EXIT THEN
   OVER > IF ." Too high " ELSE ." Too low " THEN ;
```


## Problem 8

> Using nested tests and IF … ELSE … THEN statements, write a definition
> called SPELLER which will spell out a number on the stack, from -4 to
> 4. If the number is outside this range, it will print the message “OUT
> OF RANGE.” For example:
>
> `2 SPELLER↵two ok`
> `-4 SPELLER↵negative four ok`
> `7 SPELLER↵OUT OF RANGE ok`
>
> Make it as short as possible. (Hint: The Forth word `ABS` gives the
> absolute value of a number on the stack.)

My solution (I see little point in using `ABS` to simplify such a
crude task):

```forth
: speller ( n -- )
    dup -4 = if ." negative four " drop
    else dup -3 = if ." negative three " drop
    else dup -2 = if ." negative two " drop
    else dup -1 = if ." negative one " drop
    else dup  0 = if ." zero " drop
    else dup  1 = if ." one " drop
    else dup  2 = if ." two " drop
    else dup  3 = if ." three " drop
    else dup  4 = if ." four " drop
    else ." out of range " drop 
    then then then then then then then then then
;

: test-speller ( -- )
    ." should display 'out of range ' → " -5 speller cr
    ." should display 'negative four' → " -4 speller cr
    ." should display 'zero' → " 0 speller cr
    ." should display 'one' → " 1 speller cr
    ." should display 'out of range' → " 5 speller cr
;
test-speller
```

Solution provided in the book:

```forth
: .SIGN ( n -- |n| )  DUP 0<  IF ." Negative " THEN ABS ;
: SPELLER  ( n -- )
   DUP ABS 4 > IF  ." Out of range "
      ELSE  .SIGN
      DUP  0= IF ." Zero "  ELSE
      DUP 1 = IF ." One "   ELSE
      DUP 2 = IF ." Two "   ELSE
      DUP 3 = IF ." Three " ELSE
      ." Four "
   THEN THEN THEN THEN THEN  DROP ;
```


## Problem 9

> Using your definition of `WITHIN` from Prob. 6, write another number-guessing
> game, called `TRAP`, in which you first enter a secret value, then a second
> player tries to home in on it by trapping it between two numbers, as in this
> dialogue:
>
> `0 1000 TRAP↵BETWEEN ok`
> `330 660 TRAP↵BETWEEN ok`
> `440 550 TRAP↵NOT BETWEEN ok`
> `330 440 TRAP↵BETWEEN ok`
>
> and so on, until the player guesses the answer:
>
> `391 391 TRAP↵YOU GOT IT! ok`
>
> Hint: you may have to modify the arguments to `WITHIN` so that `TRAP` does
> not say “BETWEEN” when only one of the arguments is equal to the hidden
> value.


My solution:

```forth
\ Leave “true” on the stack if low <= n <= high.
: _within ( n low high -- bool ) rot dup rot <= rot rot <= and ;
: test-_within ( -- )
    -1  -2 -3 _within assert( false = )
     1   2  3 _within assert( false = )

     0   0  0 _within assert( true = )
     0  -1  0 _within assert( true = )
    -1  -1  0 _within assert( true = )
     0  -1  1 _within assert( true = )
     0   0  1 _within assert( true = )
     2   1  3 _within assert( true = )
;
test-_within


: trap ( n n -- )
    \ Setup. We want the stack to look like this:;
    \ SECRET SECRET LOW HIGH SECRET LOW HIGH
    rot dup 2swap 2over 2over

    \ Handle the sucessful guess first.
    over = 
        if drop = 
            if ." YOU GOT IT " then 2drop 2drop

    \ Handle the other cases.
        else
            2swap drop
            _within if ." BETWEEN " else ." NOT BETWEEN " then
            2drop
        then
;

: test-trap
    0 ." should display 'BETWEEN' → " -1 1 trap drop cr
    0 ." should display 'BETWEEN' → "  0 1 trap drop cr
    0 ." should display 'NOT BETWEEN' → " 1 2 trap drop cr
    0 ." should display 'NOT BETWEEN' → " -3 -1 trap drop cr
    0 ." should display 'YOU GO IT' → " 0 0 trap cr
;
test-trap
```

Solution provided in the book:

```forth
: WITHIN ( n lo hi+1 -- flag )     OVER -  >R  -  R> U< ;
: 3DUP ( a b c -- a b c a b c )  DUP 2OVER ROT ;
: TRAP ( answer lo-try hi-try -- answer | )
   3DUP  OVER =  ROT ROT = AND IF ." You got it! "  2DROP DROP  
      ELSE  3DUP SWAP  1+  SWAP WITHIN  IF ." Between "
   ELSE ." Not between "  THEN 2DROP THEN ;
```

Yes, I was looking for something like `3dup` and did not think to just
write it myself.


## Execute this file

```txt
$ codedown forth < 2022-01-10-control-flow.md | grep . | gforth
```
