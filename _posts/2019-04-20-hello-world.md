---
title: "Hello world"
date: 2019-04-20
tags: go
---

Alan A. A. Donovan and Brian W. Kernighan, *[The Go Programming Language].*
Addison-Wesley, 2016, Chapter 1, Tutorial

[The Go Programming Language]: http://www.gopl.io/


Install language, run script from command line. Provides no REPL.
 
```sh
$ brew install golang
$ go run hello.go
```

Hello world.

```go
package main

import "fmt"

func helloWorld() {
	fmt.Println("Hello, Dünya")
}

func main() {
	helloWorld()
}
```