---
date: 2022-03-17
description: "Docker network"
image: "images/docker.svg"
title: "Docker network"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## docker network基础
1. docker 网络分为`bridge`，`host`，`none`三种
```
[root@zhugeqing ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
23bfb9b413e1   bridge    bridge    local
da3b8a887179   host      host      local
8124818e6997   none      null      local
```
2. 默认情况下，创建的container会连接到bridge上
```
[root@zhugeqing ~]# docker container run -d --rm --name linux1 ubuntu:21.04  /bin/sh -c "while true; do sleep 3600; done"
fb99caa72e20a4d0a86aa83840fe328e5dca179fc9dd0d59b9ecc4d2a9255e47

[root@zhugeqing ~]# docker container run -d --rm --name linux2 ubuntu:21.04  /bin/sh -c "while true; do sleep 3600; done"
07eab1fef9bb88386ffacd7a0a040c40a2d854daec50e5c818115d3a51bfaf3e

[root@zhugeqing ~]# docker network inspect bridge 

    "Containers": {
        "fb99caa72e20a4d0a86aa83840fe328e5dca179fc9dd0d59b9ecc4d2a9255e47": {
            "Name": "linux1",
            "EndpointID": "4ab7071e9308743a20f22ed450dde0b1acc84803019e0963149b2a1b852efb60",
            "MacAddress": "02:42:ac:11:00:06",
            "IPv4Address": "172.17.0.6/16",
            "IPv6Address": ""
        }
        "07eab1fef9bb88386ffacd7a0a040c40a2d854daec50e5c818115d3a51bfaf3e": {
            "Name": "linux2",
            "EndpointID": "e86202bc76dd0d226f8f188434b6289c96b2dd648d950b0fcc7ecb2e73c78a3d",
            "MacAddress": "02:42:ac:11:00:07",
            "IPv4Address": "172.17.0.7/16",
            "IPv6Address": ""
        },
```
3. 使用`docker network create`来创建自定义`bridge`
4. docker network driver类型为`host`和`null`的网络有且只有一个（无法再进行创建）
5. 当一个容器连接到`host`网络上的时候，可以与本机共享网络配置，且只能在创建容器的时候通过`--network`参数来指定；之后无法通过`docker network disconnet`和`docker network connect`来控制这个容器的网络配置
6. 当一个容器连接到`none`网络上的时候，将会只拥有一个本地环回地址，且只能在创建容器的时候通过`--network`参数来指定；之后无法通过`docker network disconnet`和`docker network connect`来控制这个容器的网络配置
> 可以之后在容器内部配置网络
7. `docker network create`常用参数
> -d 指定创建docker网络的 driver（实际上，只能指定bridge，而默认也是bridge）
> --gateway 指定网络的网关地址
> --subnet 指定网络的网络地址
> 使用docker network create --help 查看更多参数
## docker network相关命令
1. `docker network create -d bridge micro`
> `-d`指定网络driver，这里是`bridge`，`micro`，创建的bridge的名字
2. `docker network ls`
> 列出所有的docker网络
3. `docker network inspect micro`
> 查看docker 网络 `micro`的详情
4. `docker network connect micro linux1`
> 使容器`linux1`连接到`micro`这个docker网络上
> 可以使得同一个容器连接到多个docker 网络
5. `docker network disconnect micro linux1`
> 使容器`linux1`取消连接到`micro`这个docker网络上
6. `docker container run -d --rm --name linux1 --network micro ubuntu:21.04  /bin/sh -c "while true; do sleep 3600; done"`
> 在docker创建容器时通过使用参数`--network`指定容器所连接的docker网络
7. `docker network rm micro`
> 删除`micro`这一个docker network

## swarm overlay 网络
* 如果是使用分布式的docker环境，也就是docker swarm，那么swarm将用专门的`overlay`网络
* 在已经存在一个swarm集群的前提下，通过`docker network create -d overlay --attachable mysql` 可以创建一个mysql集群所需要的分布式系统网络