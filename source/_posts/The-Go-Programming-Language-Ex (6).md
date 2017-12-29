---
title: The Go Programming Language Ex（6）
date: 2017-12-28 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go Func

---

### Ex 5.1

Change the findlinks program to traverse the n.FirstChild linked list using recursive calls to visit instead of a loop.

<!-- more -->

```go
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) {
		fmt.Println(link)
	}
}

// visit appends to links each link found in n and returns the result.
func visit(links []string, n *html.Node) []string {

	if n == nil {
		return links
	} else {
		if n.Type == html.ElementNode && n.Data == "a" {
			for _, a := range n.Attr {
				if a.Key == "href" {
					links = append(links, a.Val)
				}
			}
		}
		links = visit(links, n.FirstChild)
		links = visit(links, n.NextSibling)
	}
	return links
}
```

```bash
$ ./ex1.7 http://www.baidu.com | ./ex5.1
/
javascript:;
javascript:;
javascript:;
/
javascript:;
https://passport.baidu.com/v2/?login&tpl=mn&u=http%3A%2F%2Fwww.baidu.com%2F
http://news.baidu.com
http://www.hao123.com
http://map.baidu.com
http://v.baidu.com
http://tieba.baidu.com
http://xueshu.baidu.com
https://passport.baidu.com/v2/?login&tpl=mn&u=http%3A%2F%2Fwww.baidu.com%2F
http://www.baidu.com/gaoji/preferences.html
http://www.baidu.com/more/
http://news.baidu.com/ns?cl=2&rn=20&tn=news&word=
http://tieba.baidu.com/f?kw=&fr=wwwt
http://zhidao.baidu.com/q?ct=17&pn=0&tn=ikaslist&rn=10&word=&fr=wwwt
http://music.baidu.com/search?fr=ps&ie=utf-8&key=
http://image.baidu.com/search/index?tn=baiduimage&ps=1&ct=201326592&lm=-1&cl=2&nc=1&ie=utf-8&word=
http://v.baidu.com/v?ct=301989888&rn=20&pn=0&db=0&s=25&ie=utf-8&word=
http://map.baidu.com/m?word=&fr=ps01000
http://wenku.baidu.com/search?word=&lm=0&od=0&ie=utf-8
//www.baidu.com/more/
//www.baidu.com/cache/sethelp/help.html
http://home.baidu.com
http://ir.baidu.com
http://e.baidu.com/?refer=888
http://www.baidu.com/duty/
http://jianyi.baidu.com/
http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11000002000001
```

### Ex 5.2

Write a function to populate a mapping from element names—p, div, span, and so on—to the number of elements with that name in an HTML document tree.

```go
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
		os.Exit(1)
	}
	m := make(map[string]int)
	for k, v := range check(m, doc) {
		fmt.Printf("%s:%d\n", k, v)
	}
}

func check(m map[string]int, n *html.Node) map[string]int {
	if n == nil {
		return m
	} else {
		if n.Type == html.ElementNode {
			m[n.Data]++
		}
		m = check(m, n.FirstChild)
		m = check(m, n.NextSibling)
	}
	return m
}
```

```bash
$ ./ex1.7 http://www.baidu.com | ./ex5.2
body:1
img:2
map:1
area:1
a:32
i:3
head:1
title:1
style:3
ul:1
meta:4
link:11
form:1
span:5
li:4
p:3
html:1
script:13
noscript:1
div:21
input:14
b:2
```

### Ex 5.5

Implement countWordsAndImages. (See Exercise 4.9 for word-splitting.)

```go
package main

import (
	"bufio"
	"fmt"
	"net/http"
	"strings"

	"golang.org/x/net/html"
)

func main() {
	url := "http://www.baidu.com"
	w, i, err := CountWordsAndImages(url)
	if err != nil {
		fmt.Println("CountWordsAndImages error: ", err)
	}
	fmt.Printf("words = %d,images = %d\n", w, i)
}

func CountWordsAndImages(url string) (words, images int, err error) {
	resp, err := http.Get(url)
	if err != nil {
		return
	}
	doc, err := html.Parse(resp.Body)
	resp.Body.Close()
	if err != nil {
		err = fmt.Errorf("parsing HTML: %s", err)
		return
	}
	words, images = countWordsAndImages(doc)
	return
}

func countWordsAndImages(n *html.Node) (words, images int) {
	if n == nil {
		return
	} else {
		if n.Type == html.ElementNode {
			if n.Data == "img" {
				images++
			}
		} else if n.Type == html.TextNode {
			scanner := bufio.NewScanner(strings.NewReader(n.Data))
			scanner.Split(bufio.ScanWords)
			for scanner.Scan() {
				words++
			}
		}
		w1, i1 := countWordsAndImages(n.FirstChild)
		words += w1
		images += i1
		w2, i2 := countWordsAndImages(n.NextSibling)
		words += w2
		images += i2
	}
	return
}
```

```bash
$ go run countwi.go
words = 2805,images = 2
```

### Ex 5.6

Modify the corner function in gopl.io/ch3/surface (§3.2) to use named results and a bare return statement.

```go
package main

import (
	"fmt"
	"math"
	"os"
)

const (
	width, height = 600, 320            // canvas size in pixels
	cells         = 100                 // number of grid cells
	xyrange       = 30.0                // axis ranges (-xyrange..+xyrange)
	xyscale       = width / 2 / xyrange // pixels per x or y unit
	zscale        = height * 0.4        // pixels per z unit
	angle         = math.Pi / 6         // angle of x, y axes (=30°)
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle) // sin(30°), cos(30°)

func main() {
	//fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
	f, err := os.Create("test.svg")
	if err != nil {
		fmt.Println("create file error ", err)
		return
	}
	fmt.Fprintf(f, "<svg xmlns='http://www.w3.org/2000/svg' "+
		"style='stroke: grey; fill: white; stroke-width: 0.7' "+
		"width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i+1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j+1)
			dx, dy := corner(i+1, j+1)
			fmt.Fprintf(f, "<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
				ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Fprintln(f, "</svg>")
}

func corner(i, j int) (sx, sy float64) {
	// Find point (x,y) at corner of cell (i,j).
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)

	// Compute surface height z.
	z := f(x, y)

	// Project (x,y,z) isometrically onto 2-D SVG canvas (sx,sy).
	sx = width/2 + (x-y)*cos30*xyscale
	sy = height/2 + (x+y)*sin30*xyscale - z*zscale
	return
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y) // distance from (0,0)
	return math.Sin(r) / r
}
```

