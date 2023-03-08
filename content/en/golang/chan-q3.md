---
date: 2023-03-08
description: "对nil、关闭的 channel、有数据的 channel，再进行读、写、关闭"
image: "/images/Go.jpg"
title: "Go Channel question"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 对一个为nil的channel进行操作
```Go:main.go
package main

import "fmt"

func main() {
    var a chan int
    fmt.Println(a == nil) // true

    // 进行一些读，写，关闭操作
	// fmt.Println(<-a) // panic
	// a <- 1          // panic
	// close(a)       // panic
}
```

## 对一个关闭的channel进行操作
```go:main.go
package main

import "fmt"

func main() {
	a := make(chan int, 1)
	a <- 100
	close(a)

	// 进行一些读，写，关闭操作
	fmt.Println(<-a) // 100
	// a <- 200      // panic
	// close(a)      // panic
}
```

## 对一个无缓冲的channel进行操作
```go:main.go
package main

func main() {
	a := make(chan int)

	// a <- 100         // 堵塞直到死锁，需要有其他goroutine来读
	// fmt.Println(<-a) // 堵塞直到死锁，需要有其他goroutine来写
	close(a)
}
```