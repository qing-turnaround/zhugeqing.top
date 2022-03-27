---
date: 2022-02-01
description: "我们总会忘记一些繁琐的事情"
image: "images/recommend_site/xingyouji.jpg"
title: "Linux设置定时任务"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- Linux
series:
- 
---

## 使用crontab指令在centos8中设置定时执行脚本

> 引出问题：当我每次完成代码或笔记编写之后，不仅需要在本地推送代码到github，还需要在云服务器上将更新的内容拉取下来。

1. 先写git pull脚本（并给予执行权限）
```shell
#!/bin/bash
cd /www/zhugeqing.top/ && git pull origin master
```

2. 设置定时执行脚本任务
* 输入`crontab –e`编辑定时任务
* 输入`50 23 * * * /root/linux_study/sh_vim/website_pull.sh`按vim操作保存
* 通过`crontab -l`查看定时任务
> crontab语法：第一个`*`表示一小时中的第几分钟（0-59），第二个`*`表示一天的第几小时（0-23），第三个`*`表示一个月的第几天（1-31），第四个`*`表示一年的第几个月（1-12），第五个`*`表示一周当中的星期几（0-7 0,7都表示星期天）
> 上例表示每天的23:50分都会执行一次shell脚本
