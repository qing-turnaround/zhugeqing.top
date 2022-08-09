---
date: 2021-12-06
description: docker swarm 部署mysql-pxc集群"
image: "images/docker.svg"
title: "docker部署 pxc集群"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 使用docker搭建 mysql PXC集群
> 使用云服务器搭建，会大致swarm集群 中容器之间无法连通，可以自建集群，或者是买 docker swarm专门的云服务，亦或者是使用虚拟机

### 先创建主节点容器
1. 保证已经存在docker swarm 集群，并且使用`docker network create -d overlay --attachable mysql-pxc`创建swarm网络

2. 在master上先创建主节点容器
```shell
docker run -d -p 13306:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=pxc      \
-e XTRABACKUP_PASSWORD=root \
-v pxc1:/var/lib/mysql \
--privileged \
--name=pxc1 \
--net=mysql-pxc \
zhugeqing/pxc
```

3. 在node1上创建从节点容器
```shell
docker run -d -p 13306:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=pxc    \
-e XTRABACKUP_PASSWORD=root \
-e CLUSTER_JOIN=pxc1  \
-v pxc2:/var/lib/mysql \
--privileged \
--name=pxc2 \
--net=mysql-pxc \
zhugeqing/pxc
```

4. 在node2上创建从节点容器
```shell
docker run -d -p 13306:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=pxc    \
-e XTRABACKUP_PASSWORD=root \
-e CLUSTER_JOIN=pxc1  \
-v pxc3:/var/lib/mysql \
--privileged \
--name=pxc3 \
--net=mysql-pxc \
zhugeqing/pxc
```
