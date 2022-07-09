---
date: 2022-03-29
description: "k8s 常用知识"
image: "images/kuber.jpg"
title: "k8s 常用知识"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## headless Service
* k8s可以为服务提供负载均衡，但是有时候开发人员希望 能够自己 来控制 负载均衡的策略，Kubernetes提供了Headless Service来实现这种功能，即不为Service设置ClusterIP（入口IP地址），也就是将Service文件中clusterIP设置为 None

## 