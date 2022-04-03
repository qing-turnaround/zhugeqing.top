---
date: 2022-04-01
description: "Channel"
image: "/images/Go.jpg"
title: "Go Channel"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## Channel 的结构
```Go:chan.go
type hchan struct {
	qcount   uint           // chan 当前里面的数据个数
	dataqsiz uint           // chan 的容量
	buf      unsafe.Pointer // 数据缓冲区
	elemsize uint16         // chan 元素 类型 的大小
	closed   uint32         // 是否已经关闭
	elemtype *_type         // chan 元素 的类型
	sendx    uint           // 记录发送者在buf中的序号
	recvx    uint           // 记录接受者在buf中的序号
	recvq    waitq          // 读取的阻塞协程队列
	sendq    waitq          // 写入的阻塞协程队列
	lock mutex              // 锁，并发保护
}

type waitq struct {
	first *sudog          // sudog 为对 goroutine数据结构的封装 
	last  *sudog
}
```

## Channel的初始化
{{< boxmd >}}
* 判断要初始化的 Channel 需分配大小是否为 0，如果为0，则只在内存中分配`hchan`结构体大小
* 当通道元素中不包含指针时，连续分配 `hchan`结构体大小和size * 元素大小（size为channel元素容量）
* 当通道元素包含指针时，需要单独分配内存空间，这样才能正常的进行垃圾回收
{{< /boxmd >}}

## Channel的写入
{{< boxmd >}}
* 先判断当前 通道是否为 nil，如果是nil，则阻塞当前协程
* 然后进行加锁，判断 通道 是否已经关闭，如果是已经关闭，则抛出 panic
* 然后判断是否有正在等待的读取协程，如果有，则从读取协程链表中获取第一个协程，并将数据复制给它，解锁，重新唤醒 goroutine
* 判断 环形buf是否有空余，如果有空余，则将数据写入环形buf，解锁，重新唤醒 goroutine
* 如果没有空余，则将 该 goroutine 放入 写入协程链表的尾部，阻塞协程，等待被再次唤醒，解锁
{{< /boxmd >}}

## Channel的读取
{{< boxmd >}}
* 先判断当前 通道是否为nil，如果为nil，则阻塞当前协程
* 然后进行加锁，判断当前通道是否已经关闭且buf为空，如果是，则 解锁，返回 通道数据类型对应的 零值，唤醒协程
* 然后先判断是否有正在等待的写入协程，有如果有，则从写入协程链表中获取第一个协程，并将它写入的数据复制到当前协程里面，解锁，等待调度
* 判断 环形buf是否有空余，如果有，则读取buf的数据，并写入当前读取的协程当中，解锁，唤醒 goroutine
* 如果没有空余，则将当前 goroutine 加入到等待读取协程链表的尾部，阻塞协程，等待被再次唤醒，解锁
{{< /boxmd >}}

## Channel的关闭
{{< boxmd >}}
* 首先判断Channel是否为nil，如果是，则抛出异常
* 上锁，判断通过是否已经 close过，如果是，则解锁，抛出异常
* 如果没有关闭过，则将 `hchan`的`closed`字段改成1，然后遍历 等待读取协程链表和等待写入协程链表，将它们的 goroutine 收集到一个队列当中， 解锁，然后遍历这个队列，挨个唤醒协程
{{< /boxmd >}}