---
date: 2022-09-17
description: "elasticsearch 初步使用"
image: "images/es.png"
title: "elasticsearch 初步使用"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- elasticsearch
series:
- 
---


## 为什么学习Elasticsearch
* Elasticsearchg高性能，容易使用，容易扩展
* Elasticsearch可以作为分布式搜索引擎，还可以用于大数据近实时分析。

## 基本概念

### 文档
* Elasticsearch 是面向文档的，文档是所有可搜索数据的最小单位
    * 比如日志文件中的日志项
    * 一个电影的具体信息
    * MP3的一首歌
* 文档被序列化程 JSON格式，保存到 Elasticsearch中
* 每个文档都有自己的 Unique ID
    * 可以自己指定ID，也可以由Elasticsearch自动生成
* 文档中可以有一些元数据
    * _index：文档所属索引名
    * _type：文档所属类型名
    * _id：文档ID
    * _source：文档的原始Json数据
    * _version：文档的版本
    * _score：文档的相应评分

### 索引
* 索引是文档的容器，是一类文档的结合
    * 每一个索引多有自己的Mapping 和 Setting
    * Mapping定义文档字段的类型
    * Setting定义不同数据的分布

### 倒排索引
* 正排索引是 从一个文档ID 找到 文档内存或单词
* 倒排索引是 从一个单词 找到文档的ID
* 倒排索引包含两个部分
    * 单词词典：记录所有文档的单词，记录单词到倒排列表的关联关系
    * 倒排列表：记录单词对应的文档集合，由倒排索引项组成
        * 倒排索引项包括 文档ID，单词出现次数，单词在语句中的位置，单词的开始结束位置


## 常用API
> put 用于指定ID的情况
### 增加一个文档

* 自动生成ID    
```shell 
post users/_doc
{
  "user": "zhugeqing"
}
```

* 指定ID，如果ID存在就报错
```shell 
put users/_doc/1?op_type=create
{
  "user": "elk"
}
```

### 更新一个文档
* 直接更新（版本加1）
```shell 
PUT users/_doc/1
{
	"user" : "elk"
}
```

### 查看一个文档
```shell
GET users/_doc/1
```

### 删除一个文档
```shell
DELETE users/_doc/1
```

### Bulk API
```shell
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```
> bulk API 支持在一次API调用中，对不同的索引进行不同的操作，支持（Index，Create，Update，Delete）
> 单条操作失败，不会影响其他操作结果
> 上例 第一个操作为 向 test索引 ID为1的文档 进行index，source为{ "field1" : "value1" }
> 第二个操作为删除 test索引中ID为2的文档

### 批量读取(mget)
> 批量读取可以减少网络连接所产生的开销，提高性能

```shell
GET /_mget
{
  "docs": [
    {
      "_index": "users",
      "_id": "1"
    },
    {
      "_index": "test",
      "_id": "2"
    }
  ]
}
```

### 批量查询(msearch)
```shell
POST /_msearch
{"index": "kibana_sample_data_ecommerce"}
{"query": {"match_all": {}}, "size": 1}
```


## 分词



