---
title: "Bitwise operations"
date: 2018-09-24
tags: c, core
published: true
---

Source: [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/), 
Data Lab (from [Lab Assignments](http://csapp.cs.cmu.edu/3e/labs.html))

The task is to implement each function, using:
- Only straightline code (no loops or conditionals)
- Only the following operators: ! ̃ & ˆ | + << >>
- No constants longer than 8 bits

Some functions include additional constraints (see below).
Constraints regarding straightline code and long constants are
relaxed for the floating-point challenges (see below).


* TOC
{:toc}


# Solutions


bitAnd()
--------
Given two 32-bit signed integers `x` and `y`, perform a bitwise
AND operation using only `~` and `|`

- Example: bitAnd(6, 5) => 4
- Legal ops: ~ |
- Max ops: 8
- Rating: 1

Explanation: De Morgan's laws state that
- NOT (A AND B) = (NOT A) OR (NOT B)
- NOT (A OR B) = (NOT A) AND (NOT B)

Therefore A AND B = NOT ((NOT A) OR (NOT B))

```c
int bitAnd(int x, int y) {
    return ~ (~x | ~y);
}
```


allEvenBits()
-------------
Given a 32-bit signed integer `x`, return 1 if all even-numbered bits
in `x` are set to 1.

- Examples:
  - allEvenBits(0xFFFFFFFE) = 0
  - allEvenBits(0x55555555) = 1
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 12
- Rating: 2

Explanation: we are not permitted to use a constant value larger 
than 8 bits, so we cannot mask directly with 0x55555555 
(0b_01010101_01010101_01010101_01010101). But we're not prohibited
from constructing such a mask from 8-bit values, for example from
0x55 (0b1010101) shifted left 24, 16, 8, and 0 places respectively.

We first mask with (our constructed) 0x55555555, to determine which
of the even bits in `x` are set to 1. Even bits set to 1 in `x` 
will be set to 1 in the product of the masking operation.

Take this result `evenBits` and perform `evenBits` XOR 0x55555555.
This will produce either an integer value greater than 0, meaning
that at least one even bit in `x` was NOT set to 1, or an integer
value of 0, meaning that all of the even bits in `x` were set to 1. 

Negate this value and we have our specified return value.

Example: 0xFFFFFFFE = 0 (false)
```
    E E E E  E E E E  E E E E  E E E E
0b_11111111_11111111_11111111_11111110 // x = 0xFFFFFFFE
0b_01010101_01010101_01010101_01010101 // mask = 0x55555555
0b_01010101_01010101_01010101_01010100 // evenBits = x & mask
                                     * // even bits not set to 1 in `x`
0b_00000000_00000000_00000000_00000001 // notAllEvenBitsAreOne = evenBits ^ mask

Example: 0x55555555 = 1 (true)
    E E E E  E E E E  E E E E  E E E E
0b_01010101_01010101_01010101_01010101 // x = 0x55555555
0b_01010101_01010101_01010101_01010101 // mask = 0x55555555
0b_01010101_01010101_01010101_01010101 // evenBits = x & mask
                                       // even bits not set to 1 in `x`
0b_00000000_00000000_00000000_00000000 // notAllEvenBitsAreOne = evenBits ^ mask
```

```c
int allEvenBits(int x) {
    int mask = 0x55 << 24 | 0x55 << 16 | 0x55 << 8 | 0x55,
        evenBits = x & mask,
        notAllEvenBitsAreOne = evenBits ^ mask;
    return !notAllEvenBitsAreOne;
}
```


bitMask()
---------
Given two 32-bit signed integers `highbit` and `lowbit` representing
a range of bits within a 32-bit pattern, return a 32-bit signed
integer in which all bits from the low bit to the high bit (inclusive) 
are set to 1.

- Example: bitMask(5, 3) = 0x38 (0b111000)
- Assume 0 <= lowbit <= 31, and 0 <= highbit <= 31
- If lowbit > highbit, then mask should be all zeroes
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 16
- Rating: 3

Explanation: we create two masks:
1. `lowBits`: all bits below the specified low bit are set to 1
2. `highBits`: all bits above the specified high bit are set to 1

Perform lowBits OR highBits. In the result, zeros occupy the range
from the specified low bit to the specified high bit. Negate this 
pattern and we have our return value.

Example: bitMask(5, 3) = 0x38 (0b111000)
```
0b_00000000_00000000_00000000_00000111 // lowBitsMask = (1 << lowbit) - 1

0b_00000011_11111111_11111111_11111111 // highBits = (1 << (31 - highbit)) - 1
0b_11111111_11111111_11111111_11000000 // highBitsMask = highBits << (highbit + 1)

0b_11111111_11111111_11111111_11000111 // resultMask = lowBitsMask | highBitsMask
0b_00000000_00000000_00000000_00111000 // ~resultMask
```

This is not an elegant solution. In the limited time available, I was 
unable to devise a solution that did not rely on subtraction. 
Working around the prohibition of the `-` operator makes it ugly.

```c
int bitMask(int highbit, int lowbit) {

    // Operator `-` is forbidden. (~1 + 1) is equivalent to - 1`.
    int x = (~1 + 1);

    int lowBitsMask = (1 << lowbit) + x;

    // Operator `-` is forbidden. (31 + (~highbit + 1)) is
    // equivalent to (31 - highbit).
    int highBits = (1 << (31 + (~highbit + 1))) + x;

    int highBitsMask = highBits << (highbit + 1),
        resultMask = lowBitsMask | highBitsMask;

    return ~resultMask;
}
```


replaceByte()
-------------
Given 32-bit signed integers `x`, `n`, and `c`, replace byte `n` in 
`x` with `c`.

- Bytes numbered from 0 (LSB) to 3 (MSB)
- Example: replaceByte(0x12345678, 1, 0xab) = 0x1234ab78
- Assume 0 <= n <= 3 and 0 <= c <= 255
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 10
- Rating: 3

Explanation:

First, we need a mask with 8 bits set to 1 in the position of the 
byte we will replace. We create it by shifting the integer byte 
position `n` (0-3) left three places (because we are working with
32-bit integers, which are 4 bytes long, with byte positions from
0-3). Then, we shift 0xff (0b11111111) by the result.

```
0xff << (0 << 3) => 0b_00000000_00000000_00000000_11111111 // mask for byte 0
0xff << (1 << 3) => 0b_00000000_00000000_11111111_00000000 // mask for byte 1
0xff << (2 << 3) => 0b_00000000_11111111_00000000_00000000 // mask for byte 2
0xff << (3 << 3) => 0b_11111111_00000000_00000000_00000000 // mask for byte 3
```

Example: `n` = 1

```
byteMask = 0xff << (1 << 3) // 0b_00000000_00000000_11111111_00000000
                                                    ^^^^^^^^
                                  BYTE 3   BYTE 2   BYTE 1   BYTE 0
```

Second, we need a mask with `c`, the replacement byte, at byte
position `n`. We create it by shifting the byte number `n` (0-3)
left three places, then shifting `c` by the result.

Example: `n` = 1, `c` = 0xab

```
cShiftedToByte = 0xab << (1 << 3) // 0b_00000000_00000000_10101011_00000000
                                                          ^^^^^^^^
                                                          BYTE 1
```

Third, we need to mask out the bits in `x` that we will replace.
We do this by masking `x` with the complement of `byteMask`.

Example: `x` = 0x12345678, `n` = 1, `c` = 0xab
```
x = 0x12345678              // 0b_00010010_00110100_01010110_01111000
~byteMask                   // 0b_00000000_00000000_00000000_11111111

xByteZeroed = x & ~byteMask // 0b_00010010_00110100_00000000_01111000
                                                    ^^^^^^^^
                                                    BYTE 1
```

Finally, we need to replace the bits at byte position `n` with `c`.
We do this by performing `xMasked OR cShiftedToByte`:

Example: `x` = 0x12345678, `n` = 1, `c` = 0xab

```
xByteZeroed = x & ~byteMask       // 0b_00010010_00110100_00000000_01111000
cShiftedToByte = 0xab << (1 << 3) // 0b_00000000_00000000_10101011_00000000
xByteZeroed | cShiftedToByte      // 0b_00010010_00110100_10101011_01111000 // 0x1234ab78
```

```c
int replaceByte(int x, int n, int c) {
    int byteMask = 0xff << (n << 3);     // 11111111 in byte position `n`
    int cShiftedToByte = c << (n << 3);  // value of `c` in byte position `n`
    int xByteZeroed = x & ~byteMask;     // `x` with 00000000 at byte `n`
    return xByteZeroed | cShiftedToByte; // `x` with `c` at byte `n`
}
```


bitParity()
-----------
Given a 32-bit signed integer `x`, return 1 if `x` contains an odd 
number of zeroes.

- Examples:
  - bitParity(5) = 0
  - bitParity(7) = 1
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 20
- Rating: 4

Explanation: each step sets `x` to the value of `x` XOR the current
value of x shifted by half as many places as remain in the bit
pattern. By retaining bits set to 1 in either of its operands, these
XOR operations resolve a bit pattern that can be masked with an
integer value of 1 to produce a return value of either 0 or 1.

Examples:
```
  0b_00000000_00000000_00000000_00000101 // x = 5
^ 0b_00000000_00000000_00000000_00000000 // x >> 16
  0b_00000000_00000000_00000000_00000101 // x = 5
^ 0b_00000000_00000000_00000000_00000000 // x >> 8
  0b_00000000_00000000_00000000_00000101 // x = 5
^ 0b_00000000_00000000_00000000_00000000 // x >> 4
  0b_00000000_00000000_00000000_00000101 // x = 5
^ 0b_00000000_00000000_00000000_00000001 // x >> 2
  0b_00000000_00000000_00000000_00000100 // x = 4
^ 0b_00000000_00000000_00000000_00000010 // x >> 1
  0b_00000000_00000000_00000000_00000110 // x = 6
& 0b_00000000_00000000_00000000_00000001 // 1
  0b_00000000_00000000_00000000_00000000 // result: 0

  0b_00000000_00000000_00000000_00000111 // x = 7
^ 0b_00000000_00000000_00000000_00000000 // x >> 16
  0b_00000000_00000000_00000000_00000111 // x = 7
^ 0b_00000000_00000000_00000000_00000000 // x >> 8
  0b_00000000_00000000_00000000_00000111 // x = 7
^ 0b_00000000_00000000_00000000_00000000 // x >> 4
  0b_00000000_00000000_00000000_00000111 // x = 7
^ 0b_00000000_00000000_00000000_00000001 // x >> 2
  0b_00000000_00000000_00000000_00000110 // x = 6
^ 0b_00000000_00000000_00000000_00000011 // x >> 1
  0b_00000000_00000000_00000000_00000101 // x = 5
& 0b_00000000_00000000_00000000_00000001 // 1
  0b_00000000_00000000_00000000_00000001 // result: 1
```

```c
int bitParity(int x) {
  x ^= (x >> 16);
  x ^= (x >>  8);
  x ^= (x >>  4);
  x ^= (x >>  2);
  x ^= (x >>  1);
  return x & 1;
}
```


tmin()
------
Return the minimum value of a two's complement integer.

- Legal ops: ! ~ & ^ | + << >>
- Max ops: 4
- Rating: 1

Explanation: minimum value representable by a 32-bit signed integer
is -2^31 or -2147483648. In binary two's complement encoding, this 
is represented as 0b_10000000_00000000_00000000_00000000. We
obtain this by left-shifting the integer value 1 by 31 places.

```c
int tmin(void) {
    return 1 << 31;
}
```


isNegative()
------------
Given a 32-bit signed integer `x`, return 1 if x < 0, or 0 otherwise.

- Example: isNegative(-1) = 1
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 6
- Rating: 2

Explanation: a signed integer in two's complement encoding is
negative if the most significant bit is set to 1. Test the most
significant bit by masking `x` with 0x80000000 (which we derive
by shifting 1 left 31 places, since we are not permitted to use
long constants). This will yield a value representing either zero
or a non-negative value. Negate its negation (in boolean terms)
using the not operator twice (`!!`) to get the specified return
value.

Example:
```
  0b_10000000_00000000_00000000_11111111 // x = -1
& 0b_10000000_00000000_00000000_00000000 // 0x80000000
  0b_10000000_00000000_00000000_00000000
```

```c
int isNegative(int x) {
    return !!(x & (1 << 31));
}
```


addOK()
-------
Given two 32-bit signed integers `x` and `y`, return 1 if `x` + `y`
can be computed without overflow, otherwise 0.

- Examples:
  - addOK(0x80000000,0x80000000) = 0
  - addOK(0x80000000,0x70000000) = 1
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 20
- Rating: 3

Explanation: an overflow will occur if the sign bits (most 
significant bits) are identical and the sign of their sum is
different.

```c
int addOK(int x, int y) {
    int signBitX = x >> 31,
        signBitY = y >> 31,
        signBitSum = (x + y) >> 31,
        xySignBitsAreSame = ~(signBitX ^ signBitY),
        signBitOfSumIsDifferent = (signBitX ^ signBitSum);
    return !(xySignBitsAreSame & signBitOfSumIsDifferent);
}
```


absVal()
--------
Given a 32-bit signed integer `x`, return the absolute value of `x`.

- Example: absVal(-1) = 1.
- You may assume -TMax <= x <= TMax
- Legal ops: ! ~ & ^ | + << >>
- Max ops: 10
- Rating: 4

Explanation: shift `x` right 31 places to create a 32-bit mask in 
which all bits are set to the value of the sign bit of `x`. This 
gives us the following:

```
0b_011111111_11111111_11111111_11111111 (0xFFFFFFFF) if x < 0
0b_00000000_00000000_000000000_00000000 (0x00000000) if x >= 0
```

Adding `x` to this mask, then performing an XOR operation with the 
sum and the mask yields the absolute value.

Example: x = -1
```
0b_00000000_00000000_00000000_11111111 // mask
0b_00000000_00000000_00000000_11111110 // x + mask
0b_00000000_00000000_00000000_00000001 // (x + mask) ^ mask
```

```c
int absVal(int x) {
    int mask = x >> 31;
    return (x + mask) ^ mask;
}
```


float_neg()
-----------
Given a 32-bit unsigned integer `uf` representing a 32-bit floating 
point value, return an unsigned integer representing the bit-level
representation of -uf.

- When argument is NaN, return argument.
- Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
- Max ops: 10
- Rating: 2

Explanation: we need to catch two special cases: 
```
0x7FC00000 // 0b_01111111_11000000_00000000_00000000
                 SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM

0xFFC00000 // 0b_11111111_11000000_00000000_00000000
                 SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM
```

In both cases, the exponent is 11111111 and the fraction is greater
than 0. This is a value that cannot be represented in 32 bits, 
and is here considered not a number. In this case, we return the 
argument. Otherwise, we simply flip the sign bit, by performing 
an XOR operation with 0x80000000.

```c
unsigned float_neg(unsigned uf) {
    unsigned NaN1 = 0x7FC00000, // 0b_01111111_11000000_00000000_00000000
             NaN2 = 0xffc00000; // 0b_11111111_11000000_00000000_00000000
    if (uf == NaN1 || uf == NaN2) return uf;
    return uf ^ 0x80000000;
}
```


float_half()
------------
Given a 32-bit unsigned integer `uf` representing a 32-bit floating
point value, return an unsigned integer representing the bit-level
representation of 0.5 * uf.

- When argument is NaN, return argument
- Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
- Max ops: 30
- Rating: 4

```c
unsigned float_half(unsigned uf) {
    // I didn't solve this one.
    return 2;
}
```
