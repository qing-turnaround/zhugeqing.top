---
date: 2022-04-01
description: "Go new与make的区别"
image: "/images/Go.jpg"
title: "Go new与make的区别"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 分配内存
{{< boxmd >}}
* Go 语言有两个分配原语，一个是`new`，一个是`make`，它们做不同的事情并适用于不同的类型，
{{< /boxmd >}}

## new
{{< boxmd >}}
* `new`它是一个分配内存的内置函数，但与其他一些语言中的同名函数不同，它不会初始化内存，它只会将其归零。也就是说， new(T)为 分配零存储 T并返回其地址，即：*T。用术语来讲的话就是：它返回一个指向新分配的类型零值的指针T。
{{< /boxmd >}}

## make 
{{< boxmd >}}
* 有了分配内存的`new`，为什么还需要有`make`呢？，答案显而易见，make只适用于三种类型：`slice`, `map`, `chan`；这三种类型与其他类型的区别就是：`它们引用的数据结构在使用之前必须被初始化！`，比如`slice`：其中就包含`cap`，`len`，`内置数组`。
* 术语化来说：`内置函数make(T, args)的用途不同于new(T)， 它只创建切片、映射和通道，并返回已初始化的（非归零值）T`
{{< /boxmd >}}