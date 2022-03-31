---
date: 2022-03-30
description: "Linux Shell脚本"
image: "images/linux.jpg"
title: "Linux Shell脚本"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- Linux
series:
- 
---

## 基本操作

* 一般在shell脚本的第一行写上`#!/bin/bash`，如果运行命令是`bash a.sh`，则这一行会被当成注释，而如果就是`./a.sh`来执行，则会解释为使用`bash`来执行。（以`#!`开头）

* shell脚本使用`#`开头来进行注释

* `shell脚本的执行方式`
{{< boxmd >}}
* `bash 1.sh`
* `./1.sh`
* `source 1.sh`
* `. 1.sh`
* 区别1：使用`./1.sh`这种方式执行，需要对文件赋予执行权限，其他不用。
* 区别2：使用`bash 1.sh`，`./1.sh`方式执行，会产生一个子进程来执行，比如sh脚本中是`cd /temp`，而在当前目录执行，执行完成之后，也不会进入`/temp`。而其他两种可以
{{< /boxmd >}}