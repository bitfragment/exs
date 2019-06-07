---
title: "Count bits"
date: 2019-06-06
tags: go, core
---

Alan A. A. Donovan and Brian W. Kernighan, *[The Go Programming Language].*
Addison-Wesley, 2016, Chapter 2, Program Structure

[The Go Programming Language]: http://www.gopl.io/

See also [Go: program structure](http://bitfragment.net/plnotes/go/program-structure/)

```go
package popcount

var pc [256]byte

func init() {
    for i := range pc {
        pc[i] = pc[i/2] + byte(i & 1)
    }
}
```

## Original version

Version given in Section 2.7: Scope.

```go
func PopCount1(x uint64) int {
    return int(pc[byte(x >> (0 * 8))] +
        pc[byte(x >> (1 * 8))] +
        pc[byte(x >> (2 * 8))] +
        pc[byte(x >> (3 * 8))] +
        pc[byte(x >> (4 * 8))] +
        pc[byte(x >> (5 * 8))] +
        pc[byte(x >> (6 * 8))] +
        pc[byte(x >> (7 * 8))])
}
```

## Exercise 2.3

> Rewrite `PopCount` to use a loop instead of a single  expression. Compare
> the performance of the two versions. (Section 11.4 shows how to compare the
> performance of different implementations systematically.)

```go
func PopCount2(x uint64) int {
    var i uint8
    var result byte
    for i = 0; i < 8; i++ {
        result += pc[byte(x >> (i * 8))]
    }
    return int(result)
}
```

## Exercise 2.4

> Write a version of `PopCount` that counts bits by shifting its argument through
> 64 bit positions, testing the rightmost bit each time. Compare its performance
> to the table-lookup version.

```go
func PopCount3(x uint64) int {
    var result uint64 = (x & 1)
    for i := 0; i < 64; i++ {
        x >>= 1
        result += (x & 1)
    }
    return int(result)
}
```

## Exercise 2.5

> The expression `x & (x - 1)` clears the rightmost non-zero bit of `x`. Write a
> version of `PopCount` that counts bits by using this fact, and assess its
> performance.

```go
func PopCount4(x uint64) int {
    count := 0
    for x > 0 {
        x &= (x - 1)
        count++
    }
    return count
}
```

## Benchmarking (test file)

```go
package popcount

import "testing"

var max32 uint64 = 4294967295

func BenchmarkPopCount1(b *testing.B) {
    for i := 0; i < b.N; i++ {
        PopCount1(max32)
    }
}

func BenchmarkPopCount2(b *testing.B) {
    for i := 0; i < b.N; i++ {
        PopCount2(max32)
    }
}

func BenchmarkPopCount3(b *testing.B) {
    for i := 0; i < b.N; i++ {
        PopCount3(max32)
    }
}

func BenchmarkPopCount4(b *testing.B) {
    for i := 0; i < b.N; i++ {
        PopCount4(max32)
    }
}
```

Output:

```bash
$ go test -bench=.
goos: darwin
goarch: amd64
BenchmarkPopCount1-4    2000000000           0.43 ns/op
BenchmarkPopCount2-4    50000000            31.2 ns/op
BenchmarkPopCount3-4    20000000            88.9 ns/op
BenchmarkPopCount4-4    50000000            34.0 ns/op
PASS
ok      _/Users/blennon/sync/desk/scratch/scratch-go/temp   6.133s
```
