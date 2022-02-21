---
date: 2022-02-01
description: "grpc"
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

3. 编写好.proto文件
```proto
syntax = "proto3"; // 指定版本（proto2, proto3）

package main; // 包名

option go_package="./test;test"; // .表示go文件存放地址， main表示生成go文件所属包名

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

4. 执行 `protoc --go_out=.  *.proto`

