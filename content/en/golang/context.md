---
date: 2022-04-01
description: "Go Context"
image: "/images/Go.jpg"
title: "Go Context"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## context 是什么

* context中文翻译为'上下文'， 其本质是用于 在goroutine之间 `传递上下文消息`，比如：取消信号，k-v键值对，超时时间，截止时间等。

* context可以协调多个goroutine 执行`取消`操作，并且还可以存储键值对，当然，它是`并发安全的`

## context的使用场景

### 传递共享数据
* 比如web 开发中，往往一个请求对应了一个goroutine，而一个请求需要执行多项流程，又会创建子goroutine来处理，但是这些流程所需要的 `验证参数`不能丢，比如一个用户请求的`token`，可以让我们在请求处理的流程中 来区分用户。

```go:main.go
package main

import (
	"context"
	"fmt"
)

func main() {
	// 创建一个根对象，
	ctx := context.Background()
	// 尝试调用help
	help(ctx)

	// 使用 WithValue创建一个 包含键值对的 子context
	childCtx := context.WithValue(ctx, "token", "user1-xx-xx-xx")
	help(childCtx)
}

func help(ctx context.Context) {
	token, ok := ctx.Value("token").(string)
	if ok {
		fmt.Println("token is ", token)
	} else {
		fmt.Println("no token")
	}
}
```

### 定时取消
* 当我们需要限制一个请求超时时间，就需要用到定时取消，因为请求已经到达了超时时间，不管程序再怎么处理，我也不需要这些处理的结果了。这时候应该要关闭 进行处理的goroutine 

```go:main.go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// 创建一个根对象，
	ctx := context.Background()

	// 使用 WithTimeout 创建一个能够定时5秒 自动取消的 子context
	childCtx, cancel := context.WithTimeout(ctx, time.Second*5)
	defer cancel()
	
	go help(childCtx)

	time.Sleep(time.Second * 20)
}

func help(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("5 seconds")
			return
		case <-time.After(time.Second * 12): // 定时12秒
			fmt.Println("12 seconds")
			return
		}
	}
}
```


### 防止goroutine泄露
* 有的时候goroutine不能如期 停止，这个时候就可以使用context 配置for-select，监听goroutinue停止信号的到达。

## contex底层原理

* context本质上是一个接口，定义了4个方法，它们都是幂等的（多次调用同一个方法，得到的结果相同）

* Deadline() 返回context的截止时间

* Done() 返回一个 只读channel，当这个channel被关闭时，说明这个contex被取消了

* Err() 返回一个错误，表示channel被关闭的原因，例如是被取消，还是超时关闭

* Value() 可以传入对应的key来获取value

* context 很大程度上是利用 通道在被关闭后 会通知所有监听它的协程这一特性来实现。在需要控制退出的子协程 使用for-select 监听context.Done()返回的通道即可，如果当前context还有子context，就会递归调用并关闭子context的通道。

* 对于使用WithCancel函数返回的子context，会有两种情况退出，一种是主动调用cancel，另一种情况是当父context退出，与其关联的所有子context都需要退出

* 对于使用WithTimeout函数返回的子context，有三种情况会退出，一种是主动调用cancel，另一种情况是当父context退出，还有一种是超时退出。