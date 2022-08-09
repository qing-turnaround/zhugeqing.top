---
date: 2022-12-07
description: "docker swam使用"
image: "images/docker.svg"
title: "docker swam使用"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 介绍
1. 弥补docker-compose无法部署在 多台机器的缺点
2. 横向扩展
3. 容器失败退出时如何新建容器确保服务正常运行
4. 尽量确保零宕机时间
5. 管理密码，Key等敏感数据

## 初步使用
1. `docker info` 查看docker engine有没有激活swarm模式，默认是`Swarm: inactive（也就是未激活）`
2. 激活swam的两种方法：`初始化一个swarm集群，自己成为manager` 和 `加入一个已经存在的swarm集群`
3. `docker swarm init` 初始化一个swarm集群，自己成为manager
> 创建swarm集群的根证书，manager节点的证书，其它节点加入集群需要的token，创建Raft数据库用于存储证书，配置，密码等数据
4. `docker swarm join ` 加入一个swarm集群
5. `docker node ls` manager 查看当前集群节点
6. `docker service create nginx:latest` 创建一个由nginx:latest镜像构成的 service
> --name web 指定名字为web
7. `docker service ls` 查看service
8. `docker service ps serviceID` 查看service的大致信息
9. `docker service update serviceID --replicas 3` 修改service 的replicas
> docker service scale serviceID=4 也可以修改replicas
10. `docker service rm serviceID` 删除service（同时容器也会被删除）
11. `docker swarm join-token worker` 重新查看加入swarm 集群的token
12. `docker service logs serviceID` 查看service日志 



## tips
* 基本上各大云服务器厂商都不支持搭建 docker swarm， 我尝试搭建，使用一台阿里云和两台腾讯云搭建docker swarm 集群，后面发现创建的分布式容器 之间无法通过创建的 overlay网络进行连接（ping通），但如果是虚拟机就没有任何问题。