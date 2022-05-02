---
date: 2022-03-18
description: "部署docker容器时，限制容器的内存"
image: "images/docker.svg"
title: "限制容器内存和管理docker占用磁盘"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 内存限制
* 使用`docker stats` 可以查看容器的内存使用状态。

* 以docker 限制 redis使用内存为例：
{{< boxmd >}}
* `docker run -d -m 500M --memory-swap -1 -name redis zhugeqing/redis redis-server`
* 上面容器启动命令的 `-m`参数指定容器的最大使用内存为`500M`
* `-- memory-swap -1`表示容器程序使用内存的受限，而可以使用的 swap 空间使用不受限制(宿主机有多少 swap 容器就可以使用多少)。
{{< /boxmd >}}


* `docker update --memory 1200M --memory-swap 1400M redis`
> 更新容器 `redis`的内存限制
> `-memory 1200M --memory-swap 1400M`表示容器能使用的内存为1200M，但swap能使用`1400M-1200M`，也就是`200M`，如果不对`--memory-swap`设置或者是设置为0，则表示能使用的swap是内存的两倍。

## 磁盘清理
* `docker system df` 查看docker磁盘占用，加入`-v`，显示更详细
* `docker container prune` 仅删除停止运行的容器。
* `docker rm -f $(docker ps -aq)` : 删除所有容器（包括停止的、正在运行的）。
* `docker image rm $(docker image ls -q)` ：删除所有镜像
* `docker volume prune ` 删除不再使用的数据卷
* `docker builder prune` 删除 build cache
* `docker system prune` 可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及 dangling 镜像（也就是无 tag 的镜像）。
* `docker system prune -a` : 清理得更加彻底，可以将没有容器使用 Docker镜像都删掉。
