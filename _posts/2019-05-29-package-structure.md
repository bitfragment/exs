---
title: "Package structure"
date: 2019-05-29
tags: go
---

Alan A. A. Donovan and Brian W. Kernighan, *[The Go Programming Language].*
Addison-Wesley, 2016, Chapter 2, Program Structure

[The Go Programming Language]: http://www.gopl.io/

See also [Go: Program structure](http://bitfragment.net/plnotes/go/program-structure/)

## scratch.go

```go
package main

import (
    "fmt"
    "./temp"
)

func main() {
    fmt.Println(tempconv.KToC(300))
}
```

## tempconv.go, in directory ./temp

```go
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64
type Kelvin float64

const (
    AbsoluteZeroC Celsius = -273.15
    BoilingC Celsius = 100
    FreezingC Celsius = 0
)

func(c Celsius) String() string {
    return fmt.Sprintf("%g°C", c)
}

func(f Fahrenheit) String() string {
    return fmt.Sprintf("%g°F", f)
}

func(k Kelvin) String() string {
    return fmt.Sprintf("%g°K", k)
}
```

## conv.go, in directory ./temp

```go
package tempconv

func CToF(c Celsius) Fahrenheit {
    return Fahrenheit(c * 9/5 + 32)
}

func FToC(f Fahrenheit) Celsius {
    return Celsius((f - 32) - 5/9)
}

func KToC(k Kelvin) Celsius {
    return Celsius(k - 273.15)
}

func KToF(k Kelvin) Fahrenheit {
    return Fahrenheit(k * 9/5 - 459.67)
}
```
