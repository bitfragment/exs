---
title: 'Word (function) definitions'
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

." thanks => " thanks cr
```


## Problem 2

> Define a word called TEN.LESS which takes a number on the stack,
> subtracts ten, and returns the answer on the stack. (Hint: you can 
> use `+.`) 

### My solution

```forth
: ten.less ( n -- n-10 ) 10 - . ;

." 20 ten.less => " 20 ten.less cr
```


## Problem 3

After entering the words in Problem 1, enter a new definition for GIVER to
print someone else’s name, then execute THANKS again. Can you explain why
THANKS still prints out the first giver’s name?

### My solution

```forth
: giver ( -- ) ." Luce " ;

." `giver` was redefined with a different person name. " cr
."   However, when `thanks` is called, you should see the original name:" cr
." thanks => " thanks

." Since `giver` is part of the definition of `thanks`, it was presumably " cr
."   compiled into `thanks`. Let's try redefining `thanks`." cr

: thanks ( -- ) ." Dear " giver ." thanks for the " gift cr ;

." thanks => " thanks
```


## Exit

```forth
bye
```



## Output

```txt
thanks => Dear Jacques thanks for the double bind of a possible impossible  
20 ten.less => 10 
`giver` was redefined with a different person name. 
  However, when `thanks` is called, you should see the original name:
thanks => Dear Jacques thanks for the double bind of a possible impossible  
Since `giver` is part of the definition of `thanks`, it was presumably 
  compiled into `thanks`. Let's try redefining `thanks`.
thanks => Dear Luce thanks for the double bind of a possible impossible  
```
