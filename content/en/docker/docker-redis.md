---
date: 2021-12-06
description: "使用docker简单部署redis"
image: "images/docker.svg"
title: "docker 部署redis"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 部署redis
1. `docker image pull zhugeqing/redis`
> 拉取镜像

2. `编写配置文件 /docker/redis/conf/redis.conf` 
{{< boxmd >}}
```
# 端口
port 6379
# 日志文件
logfile "redis.log"
# rdb文件
dbfilename "dump-redis.rdb" 
# 密码
requirepass 123456 
```
{{< /boxmd >}}


3. `docker run -d -p 6379:6379 --restart unless-stopped --name redis -v /docker/redis/conf/:/etc/redis/  -v /docker/redis/data:/data zhugeqing/redis redis-server /etc/redis/redis.conf`
> 启动容器

4. `docker container exec -it redis bash`
> 进入到容器内部

5. `redis-cli` 和 `auth 123456`来验证是否运行争取

6. `之后只需在本机修改映射的/docker/redis/conf/redis.conf，再进行docker restart redis即可更新redis配置`

