---
date: 2023-03-19
description: "Go 切片扩容，你还在背八股文吗？"
image: "/images/Go.jpg"
title: "Go 切片扩容"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 关于Go的面试中，有一个经常被问到的问题
* 面试官：`你知道Go语言切片是怎么样进行扩容的吗？`
* 背完八股文且自信满满的小明：Go切片扩容分为两种情况
    * `当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容；`
    * `当原 slice 容量 < 1024 的时候，新 slice 容量变成原来的 2 倍；`
    * `当原 slice 容量 > 1024，进入一个循环，每次容量变成原来的1.25倍,直到大于期望容量。`

* 面试官：你确定Go语言 切片扩容都是这样的吗？
* 此时小明有点不太确定，经过思索过后，回答：我当前用的版本是Go 1.17，我看的源码的扩容策略是这样的。
* 面试官也没继续问，就此打住

### 第一个问题：Go切片扩容只跟 扩容策略有关吗？
* 让我们来看以下代码
```go:main.go
package main

import (
	"fmt"
)

func main() {
	s1 := make([]int, 2, 2)
	fmt.Println(len(s1), cap(s1)) // 2, 2

	s1 = append(s1, 1, 2, 3)
	fmt.Println(len(s1), cap(s1))   // 5, ?
}
```

* 在当s1进行扩容之后，它的cap为多少呢，如果是背过八股文的，肯定知道是`5`（`当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容`）

* 当我们实际运行后 会发现，答案是`6`，为什么？[扩容源码](#源码位置补充)是在逗我？

* 让我们再细看一下源码，当完成我们所说的`八股文策略`之后，后面还有一部分源码，
```go:slice.go
......
capmem = roundupsize(uintptr(newcap))
```
* 当Go的runtime分配内存的时候，会调用roundupsize，取整内存值

* 我们需要的内存大小为 8 * 5，但是系统给我们分配了48字节，为什么？
> int 类型占8字节，需要 5个int类型

* 看一下[系统取整内存源码](#能分配多少内存大小源码)
```go:sizeclass.go
var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 24, 32, 48, 64, 80, 96, 112, .......]
```
* 当我们需要`40`字节，系统从class_to_size找到 第一个大于等于`40`字节的数，那就是`48`

* 然后因为 `48 / 8 = 6`，所以扩容结果为`6` 而不是 `5`
> 48为系统分配的字节数，8为int类型在64位机器所占字节大小

### 第一个问题：Go语言的不同版本的 Slice扩容策略都是一样的

* 先给出答案，如果你使用的版本是`Go 1.17以及之前版本`，那么就是`小明所背八股文 + 系统分配内存策略`，但是如果你使用的版本是`Go 1.18 以及当前最新版本（2023年3月19日的最新版本）`，那就有所不同

* 源码有真相，[查看完整源码，点击此处](#go-118扩容源码)
```go:slice.go
    newcap := old.cap
    // 双倍容量
	doublecap := newcap + newcap
	if cap > doublecap { // 策略1：当需要的容量 大于 双倍原容量，直接使用 需要的容量
		newcap = cap
	} else {
		const threshold = 256
		if old.cap < threshold { // 策略2：不符合策略1 并且当 原容量 小于 256时，直接使用双倍原容量 
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < cap {  
                // 策略3：当不符合策略1 且 原容量 大于等于 256时，每次原容量扩大1.25倍并加上 256 * (3/4)，直到大于等于 需要的容量
                newcap += (newcap + 3*threshold) / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```



### 总结：
* 不同的切片类型，扩容值可能是不同的，Go的runtime分配内存的时候，会调用roundupsize，取整内存值（第一个问题）
* 随着Go版本的迭代，扩容策略可能会有所改动的（第二个问题）

## 源码补充

### Go 1.17扩容源码
* [此链接](https://github.com/golang/go/blob/dev.boringcrypto.go1.17/src/runtime/slice.go) 的 162行的growslice函数

### Go 1.18扩容源码
* [此链接](https://github.com/golang/go/blob/dev.boringcrypto.go1.18/src/runtime/slice.go)的 166行的growslice函数

### 能分配多少内存大小源码
* [此链接](https://github.com/golang/go/blob/dev.boringcrypto.go1.18/src/runtime/sizeclasses.go)的93行


