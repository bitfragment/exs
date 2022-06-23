---
title: "Lissajous figures"
date: 2022-06-23
tags: go
---

Alan A. A. Donovan and Brian W. Kernighan, *[The Go Programming Language].*
Addison-Wesley, 2016, Chapter 1, Tutorial

[The Go Programming Language]: http://www.gopl.io/


## Exercise 1.12

> Modify the Lissajous server to read parameter values from the URL. For
> example, you might arrange it so that a URL like
> `http://localhost:8000/?cycles=20` sets the number of cycles to 20
> instead of the default 5. Use the `strconv.Atoi` function to convert
> the string parameter into an integer. You can see its documentation
> with `go doc strconv.Atoi`.


### My (lazy) solution

Incorporates the changes I made for [Exercise 1.6](https://bitfragment.net/exs/2019/05/08/lissajous.html).

Assumes that the query segment of the URL containes only one item and
that that item is `delay`.


```go
package main

import (
    "image"
    "image/color"
    "image/gif"
    "io"
    "log"
    "math"
    "math/rand"
    "net/http"
    "net/url"
	"regexp"
	"strconv"
)

const (
	blackIndex = 0 // first color in palette
	whiteIndex = 1 // next color in palette
	greenIndex = 2 // for Exercise 1.5
	gray1Index = 3 // for Exercise 1.6
	gray2Index = 4
	gray3Index = 5
	gray4Index = 6
)

var (
	green   = color.RGBA{0, 255, 0, 255}
	gray1   = color.RGBA{211, 211, 211, 255} // light gray
	gray2   = color.RGBA{169, 169, 169, 255} // dark gray
	gray3   = color.RGBA{105, 105, 106, 255} // dim gray
	gray4   = color.RGBA{112, 128, 144, 255} // slate gray
	palette = []color.Color{color.Black, color.White, green,
		gray1, gray2, gray3, gray4}
)

func lissajous(out io.Writer, delay int) {
	const (
		cycles  = 5     // number of complex x oscillator revolutions
		res     = 0.001 // angular resolution
		size    = 100   // image canvas covers [-size..+size]
		nframes = 64    // number of animation frames
		// delay   = 8     // delay between frames in 10ms units
	)

	freq := rand.Float64() * 3.0 // relative frequency of y oscillator
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0 // phase difference

	// Set line color index value. The third argument of 
	// `image.Palleted.SetColorIndex()` must be of type `uint8`,
	// so cast it here.
	colorChoice := uint8(3)

	// Create `nframes` images, each representing a frame of the
	// animation, storing them in the struct `anim`, interspersed
	// with delays of the value of `delay` each.
	for i := 0; i < nframes; i++ {
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)

		// Cycle the line color choice.
		colorChoice += 1
		if colorChoice == 7 {
			colorChoice = uint8(3)
		}

		// Run the two oscillators.
		for t := 0.0; t < cycles*2*math.Pi; t += res {

			// x oscillator is a sine function.
			x := math.Sin(t)

			// Frequency of y oscillator incorporates a random
			// value and the increasing value of `phase`.
			y := math.Sin(t*freq + phase)

			// Set the pixel x,y to a randomly chosen color.
			img.SetColorIndex(
				size+int(x*size+0.5),
				size+int(y*size+0.5),
				colorChoice)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}

	// Encode the sequence of frames and delays in `anim` and
	// write to output.
	gif.EncodeAll(out, &anim)
}

func handler(w http.ResponseWriter, r *http.Request) {
	// Parse the URL, using `net/url`.
	u, err := url.Parse(r.URL.String())
	if err != nil {
		panic(err)
	}
	
	// Assume the query segment of the URL contains only one item
	// and that that item is `delay`.
	re, _ := regexp.Compile("[0-9]+")
	delay, _ := strconv.Atoi(re.FindString(u.RawQuery))
	lissajous(w, delay)
}

func server() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func main() {
	server()
}
```

## Execute this file

```txt
$ codedown go < 2022-06-23-lissajous.md | grep . > /tmp/tmp.go && go run /tmp/tmp.go &
$ open http://localhost:8000/?delay=5
```
