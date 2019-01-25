---
title: "Floating-point representation"
date: 2018-09-08
tags: c
published: true
---

Source: [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/), 
practice problem 2.97

* TOC
{:toc}


## Description

Compute `(float) i`, that is, the bit-level representation of an integer
value cast to a float.


## Notes

IEEE single-precision (32-bit) floating-point word format:

`S EEEEEEEE 1.MMMMMMM MMMMMMMM MMMMMMMM`

- `S`: sign bit field
- `E`: 8-bit exponent field
- `1`: implicit (not stored) 24th bit, assumed to be 1
- `M`: 23-bit mantissa/fraction field


## Solution

```c
#include <assert.h>
#include <stdio.h>

// Bit-level representation of floating-point number.
typedef unsigned float_bits;

// I implemented three separate functions so I could test them separately.
// Obviously, everything could have been done inside `float_i2f` instead.
unsigned char sign_bit(int i);
unsigned exponent(int i);
unsigned fraction(int i);

float_bits float_i2f(int i);
```


## sign_bit()

Given a signed integer, return an unsigned char representing
the value of the sign bit in IEEE single-precision floating
point format.

```c
unsigned char sign_bit(int i) { return i < 0 ? 1 : 0; }

void test_sign_bit() {
    assert(sign_bit( 0) == 0); // sign bit should be 0
    assert(sign_bit( 1) == 0); // sign bit should be 0
    assert(sign_bit(-1) == 1); // sign bit should be 1
}
```


## exponent()

Given an integer, return an unsigned integer representing 
the 8-bit exponent in IEEE single-precision floating point
format.

Shift `i` to the right until `i` represents a value of zero, 
incrementing a counter if `i` still represents a value greater 
than zero after each shift. When `i` represents a value of zero, 
the value of the counter represents the exponent. (This is 
equivalent to repeatedly dividing `i` by 2.) Note: this returns
a value to which the bias of 127 specified by IEEE single-precision
floating-point format has yet to be added.

```c
unsigned exponent(int i) {
    unsigned expt = 0;
    while (i > 0) {
        i >>= 1;
        if (i > 0) expt++; // test before incrementing!
    }
    return expt;
}

// Ref: https://www.h-schmidt.net/FloatConverter/IEEE754.html
void test_exponent() {
    assert(exponent( 0) == 0);
    assert(exponent( 1) == 0);
    assert(exponent( 2) == 1);
    assert(exponent( 3) == 1);
    assert(exponent( 4) == 2);
    assert(exponent( 7) == 2);
    assert(exponent( 8) == 3);
    assert(exponent( 9) == 3);
    assert(exponent(15) == 3);
    assert(exponent(16) == 4);
    assert(exponent(31) == 4);
    assert(exponent(32) == 5);
    assert(exponent(2048) == 11);
    assert(exponent(16777215) == 23);
    assert(exponent(16777216) == 24);
    assert(exponent(33554432) == 25);
    assert(exponent(268435440) == 27);
}
```


## fraction()

Given a signed integer, return an unsigned integer representing the
24-bit fraction field in IEEE single-precision floating point format.
Round to even when necessary. Note: this returns a 24-bit pattern,
whose most significant bit must be masked out before it is used (the
24th bit is implicit, not stored).

```c
unsigned fraction(int i) {
    unsigned expt = exponent(i);
    unsigned fract;
    
    // printf("i: 0x%x expt: %d ", i, expt + 127); // debug

    if (expt <= 23) {
        // `expt` represents the number of significant bits in the 32-bit
        // integer `i`. To get the value that will represent the fraction 
        // in IEEE single-precision floating point, left-shift `i` by a 
        // value equivalent to the difference between 23 (the last 
        // possibly significant bit in a 24-bit fraction field) and `expt`. 
        // After the shift, the most significant bit of `i` is at the 24th 
        // place. For example, the 32-bit signed integer 0x3039 is encoded
        // in binary as follows:
        //
        // 00000000 00000000 00110000 00111001
        //                     ^ MSB
        // 
        // For 0x3039, the value of `expt` is 13, so we left-shift
        // 23 - 13 = 10 places to bring the MSB to position 24:
        // 
        // 00000000 11000000 11100100 00000000
        //          ^ new MSB
        // 
        fract = i << (23 - expt);

    } else {
        // Handle the case where `fract` has more than 24 significant bits
        // and therefore cannot be fully represented using the 24 bits of
        // the IEEE single-precision floating point fraction field. This
        // requires that we round the portion of `fract` that WILL fit in
        // a 24-bit field.
        //
        // To get the value that will represent the fraction in IEEE
        // single-precision floating point, left-shift `i` by a 
        // value equivalent to the difference between 31 (the last 
        // possibly significant bit in a 32-bit fraction field) and `expt`. 
        // (We do not want to lose any information, so we use the entire
        // 32-bit field of `i` to start with.) After the shift, the most
        // significant bit of `i` is at the 31st place.
        //
        fract = i << (31 - expt);

        // Get the rightmost 8 bits (bits 0-7) of `fract`. This is the 
        // residual part of the fraction, which will not fit into a 
        // 24-bit field. Do this by masking with 0xff, equivalent to
        // 0b_0000_0000_0000_0000_0000_0000_1111_1111.
        //
        // |---- `fract`, derived from `i` -----|
        // |   24-bit fraction field  |  residue
        // ........  ........  ........  ........
        // 31    24  23    16  15     8  7      0
        //
        unsigned residue = fract & 0xff;
        
        // printf("residue: 0x%x ", residue); // debug

        // Note: we perform this step BEFORE the rounding operation.
        // Right-shift 8 places, discarding the residue and leaving only 
        // the part of the fraction that will fit in a 24-bit field. Note
        // also: we are returning a 24-bit pattern, not the 23-bit pattern
        // specified by IEEE single-precision floating-point format, so the
        // 24th bit will have to be masked out later before using it.
        //
        // After the shift, it looks like this:
        //
        //           |   24-bit fraction field  |
        // 00000000  ........  ........  ........
        // 31    24  23    16  15     8  7      0
        //
        fract >>= 8;

        // If the residual part of the fraction represents 0x80, equivalent
        // to 0b_1000_0000 (representing exactly one-half of the previous
        // bit's place value), we must round to even.
        //
        if (residue == 0x80) {
            // printf("[round to even] "); // debug

            // Get the value of the rightmost bit of `fract`. This is 
            // the least significant bit of the part of the fraction 
            // that *will* fit in a 24-bit field.
            //
            // |---- `fract`, derived from `i` -----|
            // |-- 24-bit fraction field -|  residue
            // ........  ........  .......*  ........
            // 31    24  23    16  15     8  7      0
            //
            unsigned fract_lsb = fract & 1; // equivalent to `(fract >> 0) & 1`

            // printf("fract_lsb: %d --> ", fract_lsb); // debug

            // Round to even: if `fract_lsb` is 1, round up by adding 1 
            // to `fract`.
            if (fract_lsb) {
                // printf("round up "); // debug
                // if (fract == 0xffffff) printf("OVERFLOW! "); // debug
                fract += 1;
            } else {
                // printf("round down "); // debug
            }


        } else {
            // If the residue is less than 0b_1000_000, we round down, so don't
            // do anything here. If it is greater than 0b_1000_000, we round up
            // by adding 1 to `fract`.
            if (residue > 0x80) {
                // printf("[round up] "); // debug
                fract += 1;
            } else {
                // printf("[no rounding or round down] "); // debug
            }
        }
        
    }

    // printf("fract: 0x%x\n", fract & 0x7fffff); // debug

    return fract;
}

// Ref: https://www.h-schmidt.net/FloatConverter/IEEE754.html
void test_fraction() {
    // Exponents < 23, no rounding required.
    assert((fraction( 0) & 0x7fffff) == 0x0);
    assert((fraction( 1) & 0x7fffff) == 0x0);
    assert((fraction( 2) & 0x7fffff) == 0x0);
    assert((fraction( 3) & 0x7fffff) == 0x400000);
    assert((fraction( 4) & 0x7fffff) == 0x0);
    assert((fraction( 6) & 0x7fffff) == 0x400000);
    assert((fraction( 7) & 0x7fffff) == 0x600000);
    assert((fraction( 8) & 0x7fffff) == 0x0);
    assert((fraction( 9) & 0x7fffff) == 0x100000);
    assert((fraction(10) & 0x7fffff) == 0x200000);
    assert((fraction(2048) & 0x7fffff)== 0x0);

    // Exponents > 24, requiring rounding.
    assert((fraction(0xFFFFFF0) & 0x7fffff) == 0x7FFFFF);
    assert((fraction(0x08000001) & 0x7fffff) == 0x0);
    assert((fraction(0x080000F9) & 0x7fffff) == 0x10);
    assert((fraction(0x080000F8) & 0x7fffff) == 0x10);
    assert((fraction(0x080000E8) & 0x7fffff) == 0xE);
    assert((fraction(0x01000003) & 0x7fffff) == 0x2);
    assert((fraction(0x0FFFFFF8) & 0x7fffff) == 0x0);
}
```


## float_i2f()

Compute `(float) i`, that is, the bit-level representation
of an integer value cast to a float.

```c
float_bits float_i2f(int i) {
    // Special cases.
    if (i == 0) return 0x0;
    if (i == -0x80000000) return 0xCF000000; // -2^31

    unsigned sbit = sign_bit(i);

    // If `i` is negative, get its absolute value.
    if (sbit) i = -i;

    unsigned expt = exponent(i);
    unsigned fract = fraction(i);

    // Another special case: if the 24-bit fraction was 0xffffff
    // (0b_1111_1111_1111_1111_1111_1111), it will have overflowed the
    // 24-bit fraction field if 1 was added to the fraction to 
    // round it. So we test for that and increment the exponent here.
    //
    if (fract == 0xffffff + 1) expt += 1;

    // IEEE floating point specifies a bias of 127 for the exponent.
    expt += 127;

    // Fields we need:
    // S EEEEEEEE 1.MMMMMMM MMMMMMMM MMMMMMMM
    sbit <<= 31;
    expt <<= 23;
    fract &= 0x7fffff; // the 24th bit is implicit, so we don't need it
    
    return sbit | expt | fract;
}

// Ref: https://www.h-schmidt.net/FloatConverter/IEEE754.html
void test_float_i2f() {
    // Special cases.
    assert(float_i2f(0) == 0);
    assert(float_i2f(-0x80000000) == 0xCF000000);

    // Representable in 24 bits.
    assert(float_i2f(1) == 0x3f800000);
    assert(float_i2f(-1) == 0xBF800000);
    assert(float_i2f(2) == 0x40000000);
    assert(float_i2f(3) == 0x40400000);
    assert(float_i2f(4) == 0x40800000);
    assert(float_i2f(7) == 0x40e00000);
    assert(float_i2f(8) == 0x41000000);
    assert(float_i2f(9) == 0x41100000);
    assert(float_i2f(15) == 0x41700000);
    assert(float_i2f(16) == 0x41800000);
    assert(float_i2f(31) == 0x41f80000);
    assert(float_i2f(32) == 0x42000000);
    assert(float_i2f(64) == 0x42800000);

    // Upper bound of what is representable in 24 bits.
    // Upper bound with exponent of 23:
    assert(float_i2f(16777215) == 0x4b7fffff);
    // Upper bound of what is representable: 16777216 == 1 * 2^24
    assert(float_i2f(16777216) == 0x4b800000);

    // Rounding.
    assert(float_i2f(0x08000001) == 0x4D000000);
    assert(float_i2f(0x080000F9) == 0x4D000010);
    assert(float_i2f(0x080000F8) == 0x4D000010);
    assert(float_i2f(0x080000E8) == 0x4D00000E);
    assert(float_i2f(0x0ABCD130) == 0x4D2BCD13);
    assert(float_i2f(0x01000003) == 0x4B800002);

    // Overflows 24-bit fraction field.
    assert(float_i2f(0x0FFFFFF8) == 0x4D800000);
}
```


## Testing

```c
int main() {
    // System data types. int and float should be 32 bits.
    assert(sizeof(int) == 4);
    assert(sizeof(float) == 4);

    test_sign_bit();
    test_exponent();
    test_fraction();
    test_float_i2f();

    return 0;
}
```
