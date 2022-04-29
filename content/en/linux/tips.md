---
date: 2022-03-20
description: "释放Linux cache"
image: "images/linux.jpg"
title: "释放Linux cache"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- Linux
series:
- 
---

### 先查询一下空闲内存
* 手动清理——`echo 3 > /proc/sys/vm/drop_caches`
* `cat /proc/meminfo | grep "MemFree" | awk '{print $2}'`

### 编写脚本
> 并赋予权限
```Shell:release-Memory.sh
#! /bin/bash
#Memory小于 一般情况下的内存 时 释放Cached的内存
least=1156184
freemem=$(cat /proc/meminfo | grep "MemFree" | awk '{print $2}')
if [ $freemem -le $least ];then
    sync # 同步一下
    sync
    echo 3 > /proc/sys/vm/drop_caches # 释放
fi
```

### 设置定时任务
* 输入`crontab –e`编辑定时任务
* 输入`*/3 * * * *  /root/linux_study/release_memory.sh`按照vim操作保存
> 每隔三分钟执行一次
* 通过`crontab -l`查看定时任务