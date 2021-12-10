---
title: 'Word definitions'
date: 2021-12-08
tags: forth
src-author: Leo Brodie
src-title: '<em>Starting FORTH,</em> Chapter 1: Fundamental Forth'
src-url: https://www.forth.com/starting-forth/1-forth-stacks-dictionary/
---

## Contents
{:.no_toc}

* TOC
{:toc}


## Problem 1

> Define a word called GIFT which, when executed, will type out the name
> of some gift. For example, you might try:
>
> `: GIFT  ." Bookends " ;`
>
> Now define a word called GIVER which will print out a person’s first 
> name. Finally, define a word called THANKS which includes the new Forth 
> words GIFT and GIVER, and prints out a message something like this:
>
> `Dear Stephanie, thanks for the Bookends. ok`

### My solution

```forth
: gift ( -- ) ." double bind of a possible impossible  " ;
: giver ( -- ) ." Jacques " ;
: thanks ( -- ) ." Dear " giver ." thanks for the " gift ;
thanks
```

## Problem 2

> Define a word called TEN.LESS which takes a number on the stack,
> subtracts ten, and returns the answer on the stack. (Hint: you can 
> use `+.`) 

### My solution

```forth
: tenless ( n -- n-10 ) 10 - ;
: test-tenless assert( 20 tenless 10 = ) ;
test-tenless
```

## Problem 3

> After entering the words in Problem 1, enter a new definition for GIVER 
> to print someone else’s name, then execute THANKS again. Can you explain
> why THANKS still prints out the first giver’s name?

### My solution

Since `giver` is part of the definition of `thanks`, it was presumably
compiled into `thanks`. Let's try redefining `thanks`.

```forth
: giver ( -- ) ." Luce " ;
: thanks ( -- ) ." Dear " giver ." thanks for the " gift ;
thanks
```

## Execute this file

```txt
$ codedown forth < 2021-12-08-word-function-definitions-forth.md | grep . | gforth
Gforth 0.7.3, Copyright (C) 1995-2008 Free Software Foundation, Inc.
Gforth comes with ABSOLUTELY NO WARRANTY; for details type `license'
Type `bye' to exit
: gift ( -- ) ." double bind of a possible impossible  " ;  ok
: giver ( -- ) ." Jacques " ;  ok
: thanks ( -- ) ." Dear " giver ." thanks for the " gift ;  ok
thanks Dear Jacques thanks for the double bind of a possible impossible   ok
: tenless ( n -- n-10 ) 10 - ;  ok
: test-tenless assert( 20 tenless 10 = ) ;  ok
test-tenless  ok
: giver ( -- ) ." Luce " ; redefined giver   ok
: thanks ( -- ) ." Dear " giver ." thanks for the " gift ; redefined thanks   ok
thanks Dear Luce thanks for the double bind of a possible impossible   ok
```