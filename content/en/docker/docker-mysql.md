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
#!/bin/bash
docker pull zhugeqing/mysql:5.6
read -t 20 -p "请输入mysql的端口号（默认3306）：" port # 等待20s用户输入
port=${port:-3306} # 如果没有输入任何值则赋予变量默认值
echo "mysql端口号为：$port"

read -t 20 -p "请输入mysql的root账号密码（默认123456）：" password # 等待20s用户输入
password=${password:-123456} # 如果没有输入任何值则赋予变量默认值
echo "mysql root用户密码为：$password"

read -t 20 -p "请输入mysql容器运行的名字（默认为mysql）：" name 
name=${name:-mysql}
echo "mysql 容器运行的名字为：$name"

# 创建进行映射的文件夹
mkdir -p /docker/mysql/$name/data
mkdir -p /docker/mysql/$name/conf

# 启动容器
docker container run --name ${name} -p $port:3306 -e MYSQL_ROOT_PASSWORD=$password -d  -v ${name}-conf:/etc/mysql -v ${name}-data:/var/lib/mysql zhugeqing/mysql:5.6
 
# 建立软链接
ln -s /var/lib/docker/volumes/${name}-data/_data /docker/mysql/${name}/data
ln -s /var/lib/docker/volumes/${name}-conf/_data /docker/mysql/${name}/conf

read -t 20 -p "请输入主节点的id号（需要与从节点的id号不同，默认为1）：" id 
id=${id:-1} 
echo "主节点的id号为：$id"

cat > /docker/mysql/$name/conf/_data/mysql.cnf << EOF
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
# 主节点配置文件
[mysqld]
server-id = $id # id号
# 二进制日志
log-bin=/var/lib/mysql/mysql-bin
# 错误日志
log-error      = /var/lib/mysql/error.log 

pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql

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

docker restart ${name}
```

## 从节点配置

* `运行下面shell脚本`
```Shell:deploy:slave.sh
#!/bin/bash

docker pull zhugeqing/mysql:5.6
read -t 20 -p "请输入mysql的端口号（默认3306）：" port # 等待20s用户输入
port=${port:-3306} # 如果没有输入任何值则赋予变量默认值
echo "mysql端口号为：$port"

read -t 20 -p "请输入mysql的root账号密码（默认123456）：" password # 等待20s用户输入
password=${password:-123456} # 如果没有输入任何值则赋予变量默认值
echo "mysql root用户密码为：$password"

read -t 20 -p "请输入mysql容器运行的名字（默认为mysql）：" name 
name=${name:-mysql}
echo "mysql 容器运行的名字为：$name"

# 创建进行映射的文件夹
mkdir -p /docker/mysql/$name/data
mkdir -p /docker/mysql/$name/conf


docker container run --name ${name} -p $port:3306 -e MYSQL_ROOT_PASSWORD=$password -d  -v $name-conf:/etc/mysql -v ${name}-data:/var/lib/mysql zhugeqing/mysql:5.6

# 建立软链接
ln -s /var/lib/docker/volumes/${name}-data/_data /docker/mysql/${name}/data
ln -s /var/lib/docker/volumes/${name}-conf/_data /docker/mysql/${name}/conf

read -t 20 -p "请输入从节点的id号（需要与从节点的id号不同）（默认为2）：" id 
id=${id:-2} 
echo "从节点的id号为：$id"

cat > /docker/mysql/$name/conf/_data/mysql.cnf << EOF
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
# 主节点配置文件
[mysqld]
# id号
server-id = $id 
# relay日志
relay-log=/var/lib/mysql/relay-bin 
# 错误日志
log-error      = /var/lib/mysql/error.log 
log-slow-admin-statements = on
log-queries-not-using-indexes=on
# 开启慢查询日志
slow-query-log = 1
# 超时时间为0.03 秒
long-query-time = 0.03
# 慢查询日志路径
slow-query-log-file = /var/lib/mysql/slow.log


pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql

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




docker restart ${name}
```

* `使用MySQL客户端工具连接MySQL，下面命令均在mysql命令行使用`

* `清空以前从节点配置信息`：`reset slave;`

* `配置好关于主节点的信息，输入如下命令（对应配置进行修改）`
> `change master to master_host='主节点ip地址',master_port=主节点端口,master_user='主节点用户',master_password='主节点用户密码',master_log_file='mysql-bin.000001',master_log_pos=0;`
> 例如：`change master to master_host='127.0.0.1',master_port=3306,master_user='root',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=0;`


* `启动从节点`：`start slave;`

* `查看从节点状态`：`show slave status \G;`

