---
date: 2022-03-24
description: "解决一些在k8s中遇到的问题"
image: "images/kuber.jpg"
title: "k8s的一些问题"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- k8s
series:
- 
---

## 忘记master节点init后的join命令怎么办
* 在master节点上 执行 `kubeadm token create --print-join-command`，然后将打印出来的对应 命令在 work节点上运行
