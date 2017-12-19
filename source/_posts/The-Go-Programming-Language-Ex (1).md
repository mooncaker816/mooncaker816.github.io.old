---
title: The Go Programming Language Ex（1）
date: 2017-12-19 17:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
---

### Ex 1.1

Modify the echo program to also print os.Args[0], the name of the command that invoked it.

```go
//Modify the echo program to also print os.Args[0], the name of the command that invoked it.
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	fmt.Println("打印命令+所有参数")
	fmt.Println(strings.Join(os.Args[:], " "))
	fmt.Println("打印所有参数")
	fmt.Println(strings.Join(os.Args[1:], " "))
}
```

```bash
$ go run main.go a b c d
打印命令+所有参数
/var/folders/d8/pcxm4kx97flf9vfs3m7j_qlm0000gn/T/go-build129101527/command-line-arguments/_obj/exe/main a b c d
打印所有参数
a b c d
```

### Ex1.2

Modify the echo program to print the index and value of each of its arguments, one per line.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for i, v := range os.Args[1:] {
		fmt.Printf("第%d个参数为：%s\n", i+1, v)
	}
}
```

```bash
$ go run main.go a b c d
第1个参数为：a
第2个参数为：b
第3个参数为：c
第4个参数为：d
```

### Ex1.3

Experiment to measure the difference in running time between our po inefﬁcient versions and the one that uses strings.Join. (Section 1.6 illustrates part of the time package, and Section 11.4 shows how to write benchmark tests for systematic performance evaluation.)

```go
//比较不同版本的性能
package echo_test

import (
	"strings"
	"testing"
)

func BenchmarkEcho1(b *testing.B) {
	s1 := []string{"1", "2", "3", "4", "5"}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		s, sep := "", ""
		for _, arg := range s1 {
			s += sep + arg
			sep = " "
		}
		//fmt.Println(s)
	}
}

func BenchmarkEcho2(b *testing.B) {
	s2 := []string{"1", "2", "3", "4", "5"}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		strings.Join(s2, " ")
		//fmt.Println(strings.Join(s2, " "))
	}
}

```

```bash
$ go test -v -run="none" -bench=. -benchtime="3s" -benchmem
BenchmarkEcho1-4   	30000000	       172 ns/op	      32 B/op	       4 allocs/op
BenchmarkEcho2-4   	50000000	        86.2 ns/op	      32 B/op	       2 allocs/op
PASS
ok  	The_Go_Programming_Language_Exercises/CH1/ex1.3	9.768s
```

由此可见，`strings.Join`比`s += sep + arg`高效

###  Ex 1.4

Modify dup2 to print the names of all ﬁles in which each duplicated line occurs

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {

	m := make(map[string]map[string]int)
	files := os.Args[1:]
	//fmt.Println(os.Args[1:])
	if len(files) == 0 {
		countLines(os.Stdin, m["Stdin"])
	} else {
		for _, arg := range files {
			f, err := os.Open(arg)
			if err != nil {
				fmt.Fprintf(os.Stderr, "dup2: %v\n", err)
				continue
			}
			m[arg] = map[string]int{"": 0}
			countLines(f, m[arg])
			f.Close()
		}
	}
	for filename, mv := range m {
		for line, counts := range mv {
			if counts > 1 {
				fmt.Printf("There are %d lines (\"%s\") in file \"%s\"\n", counts, line, filename)
			}
		}
	}
}

func countLines(f *os.File, m2 map[string]int) {
	input := bufio.NewScanner(f)
	for input.Scan() {
		//fmt.Println(input.Text())
		m2[input.Text()]++
	}
	// NOTE: ignoring potential errors from input.Err()
}
```
```go
//file a
a1
a2
a3
a4
a1
a2
//file b
b1
b2
b3
b3
b3
b4
b4
b4
b5
b5
b5
b5
```

```bash
$ go run main.go a b
There are 2 lines ("a1") in file "a"
There are 2 lines ("a2") in file "a"
There are 3 lines ("b3") in file "b"
There are 3 lines ("b4") in file "b"
There are 4 lines ("b5") in file "b"
```



### Ex1.5

```go
package main

import (
	"image"
	"image/color"
	"image/gif"
	"io"
	"math"
	"math/rand"
	"os"
)

//!-main
// Packages not needed by version in book.
import (
	"log"
	"net/http"
	"time"
)

//!+main

//var palette = []color.Color{color.White, color.Black}
var palette = []color.Color{color.White, color.RGBA{0, 255, 100, 1}}//替换成绿色

const (
	whiteIndex = 0 // first color in palette
	blackIndex = 1 // next color in palette
)

func main() {
	//!-main
	// The sequence of images is deterministic unless we seed
	// the pseudo-random number generator using the current time.
	// Thanks to Randall McPherson for pointing out the omission.
	rand.Seed(time.Now().UTC().UnixNano())

	if len(os.Args) > 1 && os.Args[1] == "web" {
		//!+http
		handler := func(w http.ResponseWriter, r *http.Request) {
			lissajous(w)
		}
		http.HandleFunc("/", handler)
		//!-http
		log.Fatal(http.ListenAndServe("localhost:8000", nil))
		return
	}
	//!+main
	lissajous(os.Stdout)
}

func lissajous(out io.Writer) {
	const (
		cycles  = 5     // number of complete x oscillator revolutions
		res     = 0.001 // angular resolution
		size    = 100   // image canvas covers [-size..+size]
		nframes = 64    // number of animation frames
		delay   = 8     // delay between frames in 10ms units
	)
	freq := rand.Float64() * 3.0 // relative frequency of y oscillator
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0 // phase difference
	for i := 0; i < nframes; i++ {
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)
		for t := 0.0; t < cycles*2*math.Pi; t += res {
			x := math.Sin(t)
			y := math.Sin(t*freq + phase)
			img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5),
				blackIndex)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}
	gif.EncodeAll(out, &anim) // NOTE: ignoring encoding errors
}

//!-main

```

![替换为绿色](http://oumnldfwl.bkt.clouddn.com/out.gif)

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// Run with "web" command-line argument for web server.
// See page 13.
//!+main

// Lissajous generates GIF animations of random Lissajous figures.
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
	"os"
	"time"
)

//!-main
// Packages not needed by version in book.

//!+main

//var palette = []color.Color{color.White, color.Black}
var palette = []color.Color{color.White, color.Black, color.RGBA{0, 255, 100, 1}, color.RGBA{255, 0, 0, 1},
	color.RGBA{0, 0, 255, 1}}

const (
	whiteIndex = 0 // first color in palette
	blackIndex = 1 // next color in palette
)

func main() {
	//!-main
	// The sequence of images is deterministic unless we seed
	// the pseudo-random number generator using the current time.
	// Thanks to Randall McPherson for pointing out the omission.
	rand.Seed(time.Now().UTC().UnixNano())

	if len(os.Args) > 1 && os.Args[1] == "web" {
		//!+http
		handler := func(w http.ResponseWriter, r *http.Request) {
			lissajous(w)
		}
		http.HandleFunc("/", handler)
		//!-http
		log.Fatal(http.ListenAndServe("localhost:8000", nil))
		return
	}
	//!+main
	lissajous(os.Stdout)
}

func lissajous(out io.Writer) {
	const (
		cycles  = 5     // number of complete x oscillator revolutions
		res     = 0.001 // angular resolution
		size    = 100   // image canvas covers [-size..+size]
		nframes = 64    // number of animation frames
		delay   = 8     // delay between frames in 10ms units
	)
	freq := rand.Float64() * 3.0 // relative frequency of y oscillator
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0 // phase difference
	for i := 0; i < nframes; i++ {
		colorIndex := uint8(i % 5)
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)
		for t := 0.0; t < cycles*2*math.Pi; t += res {
			x := math.Sin(t)
			y := math.Sin(t*freq + phase)
			img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5),
				colorIndex)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}
	gif.EncodeAll(out, &anim) // NOTE: ignoring encoding errors
}

//!-main

```

![多种颜色交替](http://oumnldfwl.bkt.clouddn.com/out2.gif)

