---
date: 2022-03-07
description: "Docker volume"
image: "images/docker.svg"
title: "Docker的存储"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## Data volume
> 当容器被删除了，则数据也没有了

`Docker主要提供了两种方式做数据的持久化`
* `Data Volume`, 由Docker管理，(/var/lib/docker/volumes), 持久化数据的最好方式
* `Bind Mount`，由用户指定存储的数据具体mount在系统什么位置

### volume

1. 通过在DockerFile中加入`VOLUME [ "/app" ]`，可以使得容器中的/app目录中的数据被记录到`docker volume`中
2. 也可以在执行`docker run`时加上`-v zhugeqing:/app`来创建`docker volume`（其中`zhugeqing`为volume名字，`/app`为映射到`docker volume`的容器内目录名）
3. `docker volume ls`可以来查看所有的`volume`
4. `docker volume inspect VOLUMENAME`可以查看该`volume`的详细信息
3. `docker volume prune`可以清理不使用`volume`

### volume练习MySql

##### 准备镜像
```docker pull mysql:5.7```

#### 创建容器
[mysql镜像的详细信息](https://hub.docker.com/_/mysql?tab=description&page=1&ordering=last_updated)
* 执行 ```docker container run --name mysql5.7 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d -v mysql-data:/var/lib/mysql mysql:5.7```来创建容器

* 执行`docker volume ls`可以来查看所有的`volume`
* 执行`docker volume inspect mysql-data`来查看该容器所生成的`volume`

#### 验证
* 执行`docker container exec -it mysql5.7 mysql -u root -p`并输入密码。在数据库中创建一个名字为`zhugeqing`的数据库
* 执行`ls /var/lib/docker/volumes/mysql-data/_data`来查看生成`volume`对应的目录，发现确实也生成了一个`zhugeqing`的目录


## Bind Mount
> 提出一个问题：如果本地机器没有安装go环境，该如何使用docker来生成可执行文件到本机上？

1. 选择一个目录（比如/root），编写需要编译的go文件 /root/main.go
```Go
package main

import "fmt"

func main(){

    fmt.Println("吃掉那只青蛙！")
}
```

2. 拉取go镜像——`docker pull golang`

3. 使用go镜像运行一个容器——`docker run -it -v $(pwd):/root golang`

4. 切换到容器内的/root/目录——`cd /root`

5. 执行`go build main.go` 

6. 退出容器，发现本机的当前目录已经生成了一个可执行文件

7. 总结：使用这种方式可以让`本机的当前目前`与`容器中的指定目录`进行共享操作

