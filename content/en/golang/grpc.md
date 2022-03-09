---
date: 2022-02-01
description: "初探Go微服务"
image: "/images/Go.jpg"
title: "go-micro基础"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## protobuf
> protobuf（Google Protocol Buffer）是Google旗下的一种平台无关、语言无关、可扩展的序列化结构数据格式，很适合作为数据存储和不同语言之间相互通信的数据交换格式

* 优点： 
    * 序列化以后体积比JSON和XML格式小，便于网络传输。
    * 支持跨平台、跨语言。
    * 序列化和反序列化速度快，高于JSON的处理速度

### protobuf生成Go代码
1. 安装[protoc](https://github.com/protocolbuffers/protobuf/releases) 并加入环境变量

2. 安装[protobuf文件库](https://github.com/golang/protobuf)
> go get -u github.com/golang/protobuf/proto
> go get -u github.com/golang/protobuf/protoc-gen-go
> go get github.com/micro/micro/v2/cmd/protoc-gen-micro

3. 安装`micro工具`
> go get -v github.com/micro/micro/v2

3. 编写好.proto文件
```proto
syntax = "proto3"; // 指定版本（proto2, proto3）

package main; // proto 包名

option go_package="./test;test"; // ./test表示go文件存放地址， test表示生成go文件所属包名

service Product { // 定义的服务
  rpc AddProduct(ProductInfo) returns (ResponseProduct) {}
}

message ProductInfo { // 消息格式
  // 定义类似c语言
  int64 id = 1; // 此处的1，2代表的是字段的标识符
  string product_name = 2;
}

message ResponseProduct {
  int64 product_id = 1;
}
```

4. 执行 `protoc -I. --go_out=./ --micro_out=./ *.proto`来生成`.pb.go`和`.pb.micro.go`

* [protoc的具体使用](https://juejin.cn/post/6949927882126966820#heading-8)

### 使用docker container
* 拉取docker 镜像`docker pull zhugeqing/protoc`
* 运行一个container`docker run --rm -v $(pwd):$(pwd) -w $(pwd) zhugeqing/protoc --go_out=./ --micro_out=./  -I ./ *.proto`
> --rm表示当容器退出时自动删除容器；-v来指定bind mount；-w来指定工作目录，--go_out和--micro_out来指定生成go文件的路径；-I指定寻找proto文件的路径。



## 避雷！

`在安装micro工具包时(go get -v github.com/micro/micro/v2) 尽量选择使用go 1.14版本，在使用go 1.17版本时，会有报错信息`
```
PS D:gozhugeqing> micro help
panic: qtls.ConnectionState not compatible with tls.ConnectionState

goroutine 1 [running]:
github.com/lucas-clemente/quic-go/internal/handshake.init.1()
        D:/Code/CodeTools/GoPath/pkg/mod/github.com/lucas-clemente/quic-go@v0.14.1/internal/handshake/unsafe.go:17 +0x139 
```

* 解决：
1. 替换Go sdk版本，切换为1.14，将GoPath目录下的bin/go.exe替换成go 1.14 sdk/bin/go.exe。
2. 删除GoPath/pkg/github.com/micro/ 相关文件（此步骤只是为了以防万一，不执行应该也行）
3. 重新执行`go get -v github.com/micro/micro/v2`
4. 命令行下执行`micro help`来测试可用性

