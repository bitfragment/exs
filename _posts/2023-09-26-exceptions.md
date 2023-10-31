---
title: 'Exceptions'
date: 2023-09-26
tags: ocaml
src-author: John Whitington
src-title: '<em>OCaml from the Very Beginning</em>, Chapter 7: When Things Go Wrong'
src-url: https://johnwhitington.net/ocamlfromtheverybeginning/split12.html
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Question 1

> Write a function `smallest` which returns the smallest positive
> element of a list of integers. If there is no positive element, it
> should raise the built-in `Not_found` exception.

### My solution

Designed to use filtered lists of data as input to solve this particular
problem.

Find the smallest element of a list.

```ocaml
let smaller x y = if x <= y then x else y

let rec smallest l = match l with
  [] -> raise (Invalid_argument "Empty list")
  | h::[] -> h
  | h::x::t -> smallest ((smaller h x)::t)
```

Filter out non-positive elements.

```ocaml
let rec filter f l = match l with
  [] -> []
  | h::t -> if f h then h::(filter f t) else filter f t

let filter_pos l = match filter (fun x -> x > 0) l with
  [] -> raise Not_found (* Exception specified by the problem *)
  | h::t -> h::t
```

Solution.

```ocaml
let smallest_pos l = smallest (filter_pos l)
```

Tests.

```ocaml
let _ =
  (* Test `smallest` by itself. *)
  assert (-1 = smallest [-1; 1]);
  assert (0 = smallest [0; 1]);
  assert (1 = smallest [2; 1]);

  (* Test filtered lists including valid data. *)
  assert (1 = smallest_pos [1]);
  assert (1 = smallest_pos [1; 2]);
  assert (1 = smallest_pos [2; 1]);
  assert (1 = smallest_pos [-2; 1]);
  assert (2 = smallest_pos [2; -1]);
  assert (1 = smallest_pos [1; 2; 3]);
  assert (1 = smallest_pos [3; 2; 1]);
  assert (2 = smallest_pos [3; -2; 3; 7; -1; 0; 8; 2; -2; -6]);
;;
```

Test exceptions.

```ocaml
(* Improvised test of raised exception: https://stackoverflow.com/q/75077670 *)
let throws_exception f = match f () with 
  | exception _ -> true
  | _           -> false

let _ =
  (* Test filtered lists including no valid data. *)
  assert (throws_exception (fun () -> smallest_pos []));
  assert (throws_exception (fun () -> smallest_pos [0]));
  assert (throws_exception (fun () -> smallest_pos [-1]));
  let l = [-1; 0; -3; -5; -7; -12; 0; -1; -2] in 
    assert (throws_exception (fun () -> smallest_pos l));
  ;;
```

See exception messages displayed in the console.

```ocaml
(*
let _ = smallest [];;       (* Invalid argument *)
let _ = smallest_pos [];;   (* Not found *)
let _ = smallest_pos [0];;  (* Not found *)
let _ = smallest_pos [-1];; (* Not found *)
*)
```

### Solution provided in book

Cycles `curr` and a `found` flag to track state. An approach I made an
effort to avoid. It has a bit of an imperative smell.

```ocaml
let rec smallest_inner curr found l =
  match l with
  [] ->
    if found then curr else raise Not_found
  | h::t ->
    if h > 0 && h < curr
      then smallest_inner h true t
      else smallest_inner curr found t

let smallest l = smallest_inner max_int false l

let _ = assert (2 = smallest [3; -2; 3; 7; -1; 0; 8; 2; -2; -6]);;
```

I was relieved to find that there did not exist (at least in this book)
an obviously more compact and elegant solution than my own.

For the first time in working through this book, I find my own solution
easier to read and otherwise more elegant. However, it does atomize the
problem more extensively, which could be a disadvantage in another
context.

I also prefer the logic of treating an empty list as an
`Invalid_argument` in my solution (because there is simply no good
reason to ever use this function with an empty list), and reserving
`Not_found` for the actual processing of a list with one or more
elements. `Not_found` implies that there was something to be found, and
in my solution, that's appropriate only when it has been verified that
there is in fact something (rather than nothing) to be found.

(On the other hand, there's no intrinsic reason to treat an empty list as
invalid data. So the question of the logic here is an open one in the
end.)

## Question 2

### My solution

> Write another function `smallest_or_zero` which uses the smallest
> function but if `Not_found` is raised, returns zero.

```ocaml
let smallest_pos_or_zero l =
  try smallest (filter_pos l) with Not_found -> 0

let _ =
  assert (0 = smallest_pos_or_zero [0; -1; -2; -3]);
  assert (2 = smallest_pos_or_zero [3; -2; 3; 7; -1; 0; 8; 2; -2; -6]);
  ;;
```

### Solution provided in book

...is identical to my own.

```ocaml
let smallest_or_zero l = try smallest l with Not_found -> 0
```

## Question 3

> Write an exception definition and a function which calculates the
> largest integer smaller than or equal to the square root of a given
> integer. If the argument is negative, the exception should be raised.

### My solution

Using only `Module Stdlib`.

```ocaml
exception NegativeArgument

let sqrt_int x =
  if x < 0 then raise NegativeArgument
  else int_of_float(sqrt (float_of_int x))

let _ =
  assert (throws_exception (fun () -> sqrt_int (-1)));
  assert (0 = sqrt_int 0);
  assert (1 = sqrt_int 1);
  assert (1 = sqrt_int 2);
  assert (2 = sqrt_int 4);
  assert (2 = sqrt_int 5);
  assert (2 = sqrt_int 8);
  assert (3 = sqrt_int 9);
  ;;
```

### Solution provided in book

I did think it was odd that the problem appeared to be asking for an
implementation of `sqrt`, which is not a trivial task. That isn't the
case. Clearly I misunderstood the logic of the problem. Still, I think
its wording is susceptible to misunderstanding.

```ocaml
exception Complex

let rec sqrt_inner x n =
  if x * x > n then x - 1 else sqrt_inner (x + 1) n

let sqrt n = if n < 0 then raise Complex else sqrt_inner 1 n
```

## Question 4

> Write another function which uses the previous one, but handles the
> exception, and simply returns zero when a suitable integer cannot be
> found.

### My solution

Adapting the book's solution by wrapping `sqrt` in an exception handler.

```ocaml
let sqrt_handled n = try sqrt n with Complex -> 0

let _ = assert (0 = sqrt_handled (-1));;
```

### Solution given in book

Is identical to my own.

## Question 5

> Comment on the merits and demerits of exceptions as a method for
> dealing with exceptional situations, in contrast to returning a
> special value to indicate an error (such as `-1` for a function
> normally returning a positive number).

### My solution

Advantages of exceptions: standardized exceptions carry all of the
advantages of standardization; custom exceptions require some
interpretation but otherwise carry some of the advantages of
standardization.

Disadvantages of exceptions: they need handling; cognitive overhead of
decisions regarding the best type of exception to raise given the logic
of a problem.

Advantages of using special return values: programmer convenience?

Disadvantages of using special return values: they require more
interpretation and are non-viable without additional documentation.

### Solution provided in book

None provided.

## Execute this file

```txt
$ codedown ocaml < 2023-09-26-exceptions.md | grep . | ocaml -stdin
```
