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

# 设置redis最大内存
maxmemory 500MB
# 设置redis内存淘汰策略
maxmemory-policy volatile-lru
```
{{< /boxmd >}}


3. `docker run -d -p 6379:6379 --restart unless-stopped --name redis -v /docker/redis/conf/:/etc/redis/  -v /docker/redis/data:/data zhugeqing/redis redis-server /etc/redis/redis.conf`
> 启动容器

4. `docker container exec -it redis bash`
> 进入到容器内部

5. `redis-cli` 和 `auth 123456`来验证是否运行正确

6. `之后只需在本机修改映射的/docker/redis/conf/redis.conf，再进行docker restart redis即可更新redis配置`


## 部署redis主从

### 主节点脚本

```Shell:master.sh
#!/bin/bash

echo "正在使用redis master节点脚本！"

read -t 20 -p "请输入容器映射到主机的端口（容器内默认为6379，主机默认为6379）：" port 
port=${port:-6379}
echo "主机端口为${port}"

read -t 20 -p "请输入容器运行时的容器名字（默认为redis）：" name
name=${name:-redis}
echo "容器名字为${name}"

read -t 20 -p "请输入主节点密码（默认为123456）：" password
password=${password:-123456}
echo "主节点密码为${password}"

# 创建需要映射的目录
mkdir -p /docker/redis/$name/conf /docker/redis/$name/data


# 编写配置文件
cat > /docker/redis/$name/conf/redis.conf << EOF

# 容器内部端口
port $port
# daemonize yes 

logfile "redis-$port.log" 
dbfilename "dump-redis-$port.rdb"

# 访问redis-server密码
requirepass $password

# 主节点密码（也需要配置，不然当master无法成为新的master节点的slave节点）
masterauth $password

# 设置redis最大内存
maxmemory 500MB
# 设置redis内存淘汰策略
maxmemory-policy volatile-lru

# 开启AOF
appendonly yes
# 总是追加到AOF文件
appendfsync always
# 超过100MB就进行重写
auto-aof-rewrite-min-size 100mb
# 超过增长率就进行重写
auto-aof-rewrite-percentage 40
# 开启AOF-RDB 混合持久化
aof-use-rdb-preamble yes

# 配置慢查询日志（不超过100微秒就不会被记录）
slowlog-log-slower-than 100
# 最多记录100条，先进先出
slowlog-max-len 100
EOF

docker run -d --network host --restart unless-stopped --name $name -v /docker/redis/$name/conf:/etc/redis/  -v $name-data:/data zhugeqing/redis redis-server /etc/redis/redis.conf

# 软链接
ln -s /var/lib/docker/volumes/$name-data/_data /docker/redis/$name/data
```


### 从节点脚本
```Shell:slave.sh
#!/bin/bash
echo "正在使用redis slave节点脚本！"

read -t 20 -p "请输入容器映射到主机的端口（容器内默认为6379，主机默认为6379）：" port 
port=${port:-6379}
echo "主机端口为${port}"

read -t 20 -p "请输入容器运行时的容器名字（默认为redis）：" name
name=${name:-redis}
echo "容器名字为${name}"

read -t 20 -p "请输入主节点的ip地址（默认为127.0.0.1）：" masterip
masterip=${masterip:-127.0.0.1}
echo "主节点的ip为${masterip}"

read -t 20 -p "请输入主节点的端口（默认为6379）：" masterport
masterport=${masterport:-6379}
echo "主节点的端口为${masterport}"

read -t 20 -p "请输入主节点密码（默认为123456）：" password
password=${password:-123456}
echo "主节点密码为${password}"

# 创建需要映射的目录
mkdir -p /docker/redis/$name/conf /docker/redis/$name/data


# 编写配置文件
cat > /docker/redis/$name/conf/redis.conf << EOF

# 容器内部端口
port $port
# daemonize yes 

logfile "redis-$port.log" 
dbfilename "dump-redis-$port.rdb"

# 访问redis-server密码
requirepass $password
# 主节点密码
masterauth $password

# 配置主节点 host
slaveof $masterip $masterport

# 设置redis最大内存
maxmemory 500MB
# 设置redis内存淘汰策略
maxmemory-policy volatile-lru

# 开启AOF
appendonly yes
# 总是追加到AOF文件
appendfsync always
# 超过100MB就进行重写
auto-aof-rewrite-min-size 100mb
# 超过增长率就进行重写
auto-aof-rewrite-percentage 40
# 开启AOF-RDB 混合持久化
aof-use-rdb-preamble yes


# 配置慢查询日志（不超过100微秒就不会被记录）
slowlog-log-slower-than 100
# 最多记录100条，先进先出
slowlog-max-len 100
EOF

docker run -d --network host --restart unless-stopped --name $name -v /docker/redis/$name/conf:/etc/redis/  -v $name-data:/data zhugeqing/redis redis-server /etc/redis/redis.conf

# 软链接
ln -s /var/lib/docker/volumes/$name-data/_data /docker/redis/$name/data
```

## 部署redis-sentinel

```Shell:sentinel.sh
#!/bin/bash

echo "当前正在使用部署redis-sentinel的脚本!"

read -t 20 -p "请输入容器映射到主机的端口（容器内默认为26379，主机默认为26379）：" port
port=${port:-26379}
echo "主机端口为${port}"

read -t 20 -p "请输入容器运行时的容器名字（默认为redis-sentinel）：" name
name=${name:-redis-sentinel}
echo "容器名字为${name}"

read -t 20 -p "请输入sentinel监控主节点的名字（默认为redis-master）：" mastername
mastername=${mastername:-redis-master}
echo "主节点的名字为${mastername}"


read -t 20 -p "请输入主节点的ip地址（默认为127.0.0.1）：" masterip
masterip=${masterip:-127.0.0.1}
echo "主节点的ip为${masterip}"

read -t 20 -p "请输入主节点的端口（默认为6379）：" masterport
masterport=${masterport:-6379}
echo "主节点的端口为${masterport}"

read -t 20 -p "请输入主从的密码（默认为123456）：" masterpasswd
masterpasswd=${masterpasswd:-123456}
echo "主从节点的密码为${masterpasswd}"

read -t 20 -p "请输入sentinel选举最小投票数（默认为1）：" vote
vote=${vote:-1}
echo "最小投票数为${vote}"

read -t 20 -p "请输入sentinel密码（默认为123456）：" password
password=${password:-123456}
echo "sentinel密码为为${password}"



# 创建需要映射的目录
mkdir -p /docker/redis/$name/conf /docker/redis/$name/data

cat > /docker/redis/$name/conf/sentinel.conf << EOF
# 端口
port $port

# 密码
requirepass $password

logfile "redis-sentinel-$port.log" 

sentinel monitor $mastername $masterip $masterport $vote

# 这个配置项指定了需要多少失效时间，一个master才会被这个sentinel主观地认为是不可用的。 单位是毫秒，默认为30秒      
sentinel down-after-milliseconds $mastername 30000 

sentinel auth-pass $mastername $masterpasswd

# 指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步（默认为1）
sentinel parallel-syncs $mastername 1

# 故障转移时间
sentinel failover-timeout $mastername 180000
EOF


docker run -d --network host --restart unless-stopped --name $name -v /docker/redis/$name/conf:/etc/redis/ -v $name-data:/data zhugeqing/redis redis-sentinel /etc/redis/sentinel.conf

# 软链接
ln -s /var/lib/docker/volumes/$name-data/_data /docker/redis/$name/data
```

## 删除上述操作脚本

```Shell:rm-redis.sh
#!/bin/bash
read -t 20 -p "请输入docker容器的名字：" name
docker rm -f $name
docker volume rm $name-data
rm -rf /docker/redis/$name
```