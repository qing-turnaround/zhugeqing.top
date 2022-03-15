---
date: 2022-03-15
description: "三种Go实现方式"
image: "/images/Go.jpg"
title: "Go语言实现生产者消费者模型"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 无缓冲的生产者消费者模型
```Go
package main

import "fmt"

func producer(out chan<- int) {
	for i := 0; i < 10; i++ {
		data := i * i
		fmt.Println("生产者生产数据:", data)
		out <- data // 缓冲区写入数据
	}
	close(out) //写完关闭管道
}

func consumer(in <-chan int) {
	// 没有数据就阻塞
	for data := range in {
		fmt.Println("消费者得到数据：", data)
	}

}

func main() {
	ch := make(chan int) //无缓冲channel
	go producer(ch)      // 子go程作为生产者
	consumer(ch)         // 主go程作为消费者
}
```

## 有缓冲的生产者消费者模型
```Go
package main

import "fmt"

func producer(out chan<- int) {
	for i := 0; i < 10; i++ {
		data := i * i
		fmt.Println("生产者生产数据:", data)
		out <- data // 缓冲区写入数据
	}
	close(out) //写完关闭管道
}

func consumer(in <-chan int) {
	// 没有数据就阻塞
	for data := range in {
		fmt.Println("消费者得到数据：", data)
	}

}

func main() {
	ch := make(chan int, 5) //有缓冲channel
	go producer(ch)      // 子go程作为生产者
	consumer(ch)         // 主go程作为消费者
}
```



## 多消费者多生产者模型
```Go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var cond sync.Cond // 条件变量来协同生产者消费者

func producer(id int, out chan<- int) {
	for {
		cond.L.Lock()
		for len(out) == 5 {
			cond.Wait() // 产品区满等待消费者消费
		}
		product := rand.Intn(1000)
		fmt.Printf("编号为%d的生产者生成的产品为%d\n", id, product)
		out <- product
		cond.L.Unlock()
		cond.Signal()                      // 生产完成，唤醒消费者协程
		time.Sleep(100 * time.Millisecond) //等待10毫秒，给其他生产者协程机会
	}
}

func consumer(id int, in <-chan int) {
	for {
		cond.L.Lock()
		for len(in) == 0 {
			cond.Wait() // 产品区为空等待生产者生产
		}
		fmt.Printf("编号为%d的消费者消费的产品为%d\n", id, <-in)
		cond.L.Unlock()
		cond.Signal()                      // 消费完成，唤醒生成者协程
		time.Sleep(100 * time.Millisecond) //等待10毫秒，给其他消费者协程机会
	}

}

func main() {
	quit := make(chan bool)      // 阻塞主协程
	product := make(chan int, 5) // 产品区
	// 初始化cond结构体的互斥锁（分配内存空间，不然会报错）
	cond.L = new(sync.Mutex)
	for i := 0; i < 5; i++ {
		go producer(i, product) // 开启5个生产者
	}
	for i := 0; i < 5; i++ {
		go consumer(i, product) // 开启5个消费者
	}

	<-quit
}
```