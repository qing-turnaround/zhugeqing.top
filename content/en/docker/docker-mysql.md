---
date: 2021-12-06
description: "使用docker部署简单mysql集群主从复制"
image: "images/docker.svg"
title: "docker 部署mysql集群"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---


## 主节点配置

* `运行下面shell脚本`
```Shell:deploy-master.sh
docker pull zhugeqing/mysql:5.6

read -t 5 -p "请输入mysql的端口号：（默认3306）" port # 等待5s用户输入
port=${port:-3306} # 如果没有输入任何值则赋予变量默认值
echo "mysql端口号为：$port"

read -t 5 -p "请输入mysql的root账号密码：（默认123456）" password # 等待5s用户输入
password=${password:-123456} # 如果没有输入任何值则赋予变量默认值
echo "mysql root用户密码为：$password"

docker container run --name mysql -p port:3306 -e MYSQL_ROOT_PASSWORD=123456 -d  -v mysql-etc:/etc/mysql -v /var/lib/mysql:/var/lib/mysql zhugeqing/mysql:5.6


read -t 5 -p "请输入主节点的id号（需要与从节点的id号不同）：（默认为1）" id 
id=${id:-1} 
echo "主节点的id号为：$id"

cat >> /var/lib/docker/volumes/mysql-etc/_data/mysql.cnf << EOF
# 主节点配置文件
[mysqld]
server-id = $id # id号
log-bin=/var/lib/mysql/mysql-bin
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error      = /var/log/mysql/error.log

symbolic-links=0
character_set_server=utf8
init_connect='SET NAMES utf8'
max_allowed_packet = 20M

[mysql]
default-character-set = utf8

[mysql.server]
default-character-set = utf8

[mysqld_safe]
default-character-set = utf8

[client]
default-character-set = utf8
EOF

docker restart mysql
```

## 从节点配置

* `运行下面shell脚本`
```Shell:deploy:slave.sh
docker pull zhugeqing/mysql:5.6

read -t 5 -p "请输入mysql的端口号：（默认3306）" port # 等待5s用户输入
port=${port:-3306} # 如果没有输入任何值则赋予变量默认值
echo "mysql端口号为：$port"

read -t 5 -p "请输入mysql的root账号密码：（默认123456）" password # 等待5s用户输入
password=${password:-123456} # 如果没有输入任何值则赋予变量默认值
echo "mysql root用户密码为：$password"

docker container run --name mysql -p port:3306 -e MYSQL_ROOT_PASSWORD=123456 -d  -v mysql-etc:/etc/mysql -v /var/lib/mysql:/var/lib/mysql zhugeqing/mysql:5.6


read -t 5 -p "请输入从节点的id号（需要与从节点的id号不同）：（默认为2）" id 
id=${id:-2} 
echo "从节点的id号为：$id"

cat >> /var/lib/docker/volumes/mysql-etc/_data/mysql.cnf << EOF
# 主节点配置文件
[mysqld]
server-id = $id # id号
relay-log=/var/lib/mysql/relay-bin # relay日志
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error      = /var/log/mysql/error.log

symbolic-links=0
character_set_server=utf8
init_connect='SET NAMES utf8'
max_allowed_packet = 20M

[mysql]
default-character-set = utf8

[mysql.server]
default-character-set = utf8

[mysqld_safe]
default-character-set = utf8

[client]
default-character-set = utf8
EOF

docker restart mysql
```

* `使用MySQL客户端工具连接MySQL，下面命令均在mysql命令行使用`

* `配置好关于主节点的信息，输入如下命令（对应配置进行修改）`
> `change master to master_host='主节点ip地址',master_port=主节点端口,master_user='主节点用户',master_password='主节点用户密码',master_log_file='mysql-bin.000001',master_log_pos=0;`
> 例如：`change master to master_host='127.0.0.1',master_port=3306,master_user='root',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=0;`

* `启动从节点`：`start slave;`

* `查看从节点状态`：`show slave status`

