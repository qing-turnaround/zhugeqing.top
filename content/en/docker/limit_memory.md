---
date: 2021-12-06
description: "部署docker容器时，限制容器的内存"
image: "images/docker.svg"
title: "限制容器内存"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

* 使用`docker stats` 可以查看容器的内存使用状态。

* 以docker 限制 redis使用内存为例：
{{< boxmd >}}
* `docker run -d -m 500M -- memory-swap -1 -name redis zhugeqing/redis redis-server`
* 上面容器启动命令的 `-m`参数指定容器的最大使用内存为`500M`
* `-- memory-swap -1`表示容器程序使用内存的受限，而可以使用的 swap 空间使用不受限制(宿主机有多少 swap 容器就可以使用多少)。
{{< /boxmd >}}


* `docker update --memory 1200M --memory-swap 1400M redis`
> 更新容器 `redis`的内存限制
> `-memory 1200M --memory-swap 1400M`表示容器能使用的内存为1200M，但swap能使用`1400M-1200M`，也就是`200M`，如果不对`--memory-swap`设置或者是设置为0，则表示能使用的swap是内存的两倍。
