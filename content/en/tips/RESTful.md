---
date: 2021-11-12
description: "简单理解RESTful"
image: "/images/xingyouji.jpg"
title: "RESTful"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- tips
series:
- 
---

## REST

`REST`为`Representational State Transfer`的缩写，中文翻译为`表现层状态转换`，本质是一种架构的原则，如果一个架构符合`REST`原则，就称它为`RESTful`架构。
> 严格来说，`REST`是`设计风格`而不是`标准`

## Representation（表现层）

`资源（html，xml，txt，json，png，jpg...）`是信息实体，可以拥有多种表现的形式，而`资源所呈现的形式`，就叫做资源的`表现层（Representation）`

## State Transfer（状态转换）

在`客户端和服务器的通信的过程中`，客户端通过使用`HTTP协议`的`操作（Get，Put，Post，Delete）`来改变服务端资源。`让服务器端发生"状态转化"（State Transfer）`

## RESTful API
* 直观简短的资源地址：URI，比如：http://example.com/resources。
* 传输的资源：Web服务接受与返回的互联网媒体类型，比如：JSON，XML，YAML等。
* 对资源的操作：Web服务在该资源上所支持的一系列请求方法（比如：POST，GET，PUT或DELETE）。

