---
title: golang-module
lastmod: 2025-05-09T16:43:23+08:00
date: 2025-05-09T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-module.html
toc: true
---

## go module

### fmt

```go
package main

import "fmt"

func main() {
	var f float32
	var g float64
	f = 1.4444444123231
	g = 3.45554633

	fmt.Printf("f=%[1]f, g=%.2[2]f, g=%[2]g,f=%[1]e", f, g)
}
```

