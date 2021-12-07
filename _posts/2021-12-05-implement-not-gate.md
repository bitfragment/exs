---
title: Implement Not gate
date: 2021-12-05
tags: hdl, core
src-author: Nisan and Schocken
src-title: 'The Elements of Computing Systems: Building a Computer from First Principles'
src-url: https://www.nand2tetris.org/book
---

## Contents
{:.no_toc}

* TOC
{:toc}

## Description

Project 01, book section 1.5

> Implement all the logic gates presented in the chapter. The only
building blocks that you can use are the primitive Nand gates and
the composite gates that you will gradually build on top of them.

Chip #1 of 15: implement Not gate (converter or inverter)


## Specification

* Inputs: `in`  
* Outputs: `out`
* Function: `If in=0 then out=1 else out=0`


## Reference

* [NOT gate](https://en.wikipedia.org/wiki/NOT_gate)
* [NOT from NAND](https://en.wikipedia.org/wiki/File:NOT_from_NAND.svg)


## Implementation

The output of a Nand gate is true if one of its inputs is false.

The Nand gate supplied with the `nand2tetris` hardware simulator has
two inputs.

Wire the single input of a Not gate to both inputs of a Nand gate.
The Not gate passes a single value to both inputs of the Nand gate.

If that value is 0, the Nand gate returns 1, inverting the Not gate's
input. Likewise, if that value is 1, the Nand gate returns 0, an
inversion of the Not gate's input:

```
Not input | Nand input A | Nand input B | Nand output
0         | 0            | 0            | 1
1         | 1            | 1            | 0
```

(Other implementations are possible.)
 
File `Not.hdl`:

```hdl
CHIP Not {
    IN in;
    OUT out;

    PARTS:
    Nand(a = in, b = in, out = out);
}
```


## Testing

File `Not.tst`:

```txt
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/01/Not.tst

load Not.hdl,
output-file Not.out,
compare-to Not.cmp,
output-list in%B3.1.3 out%B3.1.3;

set in 0,
eval,
output;

set in 1,
eval,
output;

```

Test result in file `Not.out`:

```txt
|  in   |  out  |
|   0   |   1   |
|   1   |   0   |
```

File `Not.cmp`:

```txt
|  in   |  out  |
|   0   |   1   |
|   1   |   0   |
```
