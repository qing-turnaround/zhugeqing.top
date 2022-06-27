---
date: 2022-03-17
description: "Docker compose"
image: "images/docker.svg"
title: "Docker compose"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 基本使用
1. `docker-compose up` 创建并启动docker-compose.yml的容器和其他的资源
> 加入 -d 可以后台运行，加入--build 可以运行之前构建镜像
2. `docker-compose stop` 停止docker-compose.yml 定义的容器
3. `docker-compose start` 启动 docker-compose.yml 定义的容器
> docker-compose restart 就是重新启动 
4. `docker-compose rm` 删除docker-compose.yml 定义的容器
5. `docker-compose down` 停止并删除 docker-compose.yml 定义的容器
6. `docker-compose ps` 查看 docker-compose.yml 定义的容器的运行情况
7. `docker-compose build` 对 docker-compose.yml 指定的 dockerfile 构建镜像
> 在build 后面可以指定 镜像名字

## docker-compose的安装
* `curl -L "https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
* `chmod 744 /usr/local/bin/docker-compose`
> 赋予命令执行权限
* `docker-compose --version`
> 测试是否安装成功

## compose 文件的结构
> docker compose文件的语法说明 https://docs.docker.com/compose/compose-file/
```yaml:docker-compose.yml
version: "3.8" # version是必须指定的，而且总是位于文件的第一行。 
# 它定义了Compose文件格式（主要是API）的版本。建议使用最新版本。

services: # 容器
  servicename: # 服务名字，这个名字也是内部 bridge网络可以使用的 DNS name
    build: # 可选，相当于执行docker build，这个选项填 dockerfile的路径
    image: # 镜像的名字
    command: # 可选，如果设置，则会覆盖默认镜像里的 CMD命令
    environment: # 可选，相当于 docker run里的 --env
    volumes: # 可选，相当于docker run里的 -v
    networks: # 可选，相当于 docker run里的 --network
    ports: # 可选，相当于 docker run里的 -p
  servicename2:

volumes: # 可选，相当于 docker volume create

networks: # 可选，相当于 docker network create
```

### 例如
```yaml:docker-compose.yml
version: "3.8"

services:
  flask-demo:
    image: flask-demo:latest
    environment:
      - REDIS_HOST=redis-server
    networks:
      - demo-network
    ports:
      - 8080:5000

  redis-server:
    image: redis:latest
    networks:
     - demo-network

networks:
   demo-network:
      ipam:
        driver: default
        config: 
         -subnet: "172.16.200.0/24"
```

## docker-compose 水平扩展

* `docker-compose up -d --scale flask=3` 扩展 servicename为 flask的容器为3 
> 
