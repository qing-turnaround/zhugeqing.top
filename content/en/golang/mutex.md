---
date: 2022-04-01
description: "Go Mutex以及优化"
image: "/images/Go.jpg"
title: "并发原语之Mutex"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 使用示例
{{< boxmd >}}
* 开启10个goroutine，每一个goroutine进行计数1000000次
{{< /boxmd >}}
```Go:main.go
package main

import (
	"sync"
)

type Counter struct {
	count int
	mu sync.Mutex
}

func main() {
	var wg sync.WaitGroup
	counter := &Counter{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			for j := 0; j < 1000000; j++ {
				counter.mu.Lock()
				counter.count++
				counter.mu.Unlock()
			}
			wg.Done()
		}()
	}
	wg.Wait()
	println("计数得到的结果为：", counter.count)
}
```

## Mutex底层实现
> sync.Mutex 设计考量，效率优先（正常模式） 和 兼顾公平（饥饿模式）
> 好几年前的理解，有空更新
* Mutex一共有两个字段，一个为`state`，为int32类型，其中有三个比特为`mutexLocked`（记录这个锁是否被持有），`mutexWoken`（是否有唤醒的 goroutine），`mutexStarving`（记录是否处于饥饿模式），其余比特用于记录等待此锁的 goroutine 数（最大为`1<<(32-3)-1`个）
* 在正常模式下，当一个 goroutine 未获得锁时会先进行自旋几次，如果还是没有获得到锁，就加入等待队列的尾部，等待队列会按照FIFO的顺序排队。当锁处于未被持有的状态时，等待队列的第一个 goroutine会被唤醒，然后需要与后来的处于自旋状态的 goroutine 进行竞争，这时，后来的 goroutine 会更有优势获得锁，一方面是它们仍在cpu上运行，另一方面是处于自旋状态的 goroutine 可以有多个，而被唤醒的 goroutine 只有一个，所以被唤醒的goroutine 大概率会失败。然后这个 goroutine 会被重新插入到队列的首部。但是当一个 goroutine 加锁等待时间超过了 1ms 以后，Mutex会进入饥饿模式。
* 在饥饿模式下，Mutex的所有权会从执行Unlock的 goroutine 直接传递给等待队列的头部的 goroutine，后来的goroutine也不会再进行自旋操作，而是直接加入等待队列的尾部，当等待队列的头部goroutine 加锁等待时间小于 1ms 或者是等待队列为空时，Mutex会重新切换为正常模式


## 优化锁

### 缩小临界区

* 通常我们可能在使用`mutex.Lock()` 之后加上 `defer mutex.Unlock()`, 但这可能让临界区代码增大，从而降低性能，可以在锁 保护完一段 逻辑之后立马释放锁（但是需要注意以后修改代码，不要遗漏解锁操作）


### 减小锁的粒度——分段锁

* 一把大锁 ——》多把小锁，空间换时间的思想（比如我们为了并发安全的map，可能会使用分段锁Map）

### 让锁做读写分离

* sync.Mutex -> sync.RWMutex
* 使用sync.Map

### 无锁化

* 可以使用一些原子操作代替锁


## 锁的一些避坑

* 锁是不能拷贝的（我们传递锁的话，需要传递指针）
* sync.Mutex是不可重入的（再次加锁之前，必须执行释放锁的操作）
* atomic.Value 只保证`存储对象时`是并发安全的，要么存放只读对象，要么对 对象的操作是并发安全的
* `我们可以使用go run -race ...` 来检测并发竞争问题