---
date: 2022-09-18
description: "Elasticsearch Mapping 学习"
image: "images/es.png"
title: "Mapping 学习"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- elasticsearch
series:
- 
---

## Mapping是什么
* Mapping 类似数据库的 schema的定义，可以用于
    * 定义索引中的字段名称
    * 定义字段的数据类型
    * 字段，倒排索引的相关配置（比如分词）
* Mapping 可以分为Dynamic Mapping（动态Mapping） 和 Explicit Mapping（显式Mapping）


## Dynamic Mapping
* 在写入文档时候，如果索引不存在，会自动创建索引
* Dynamic Mapping可以让我们不需要手动定义Mapping，Elasticsearch会自动推断出字段的类型
* 但是有时候类型的推断并不符号我们的预期，从而导致一些查询无法正常进行

### 类型的自动识别
* 字符串(JSON)
    * 匹配日期格式，设置成Date
    * 匹配数字，设置为float或者long
    * 匹配字符串，设置为Text，并且增加keywork子字段
* 布尔值
    * 匹配 boolean
* 浮点数
    * 匹配 float
* 整数
    * 匹配 long
* 对象
    * 匹配 Object
* 数组
    * 由数组中第一个非空数值的类型决定
* 空值
    * 忽略

### 设置Mapping 的dynamic
| 设置的值         | true        | false       | strict      |
| ----------------|-------------|-------------|-------------|
| 文档是否能索引   | Yes       | Yes       | No       |
| 字段是否能索引   | Yes        | No       | No       |
| mapping是否更新| YES        | No       | NO       |

* 当dynamic设置成 true 时，有新增字段的写入，Mapping也同时被更新（默认为true）
* 当dynamic设置成 false时，mapping不会被更新，新增字段的数据无法被索引，但是信息可以存在_source中（也就是可以被其他的字段查询到）
* 当dynamic设置为 strict时，文档写入失败
* 对于索引已经存在的字段，无法修改字段的定义，除非使用Reindex API 重建索引


## 显式Mapping

### 控制字段是否被索引
```shell
#设置 index 为 false
PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "text",
          "index": false
        }
      }
    }
}

PUT users/_doc/1
{
  "firstName": "qing",
  "lastName": "zhuge",
  "mobile" : "123"
}
# 下面无法的mobile字段无法被search
GET users/_search?q=mobile:123 
```

### copy_to

```shell
#设置 Copy to
DELETE users
PUT users
{
  "mappings": {
    "properties": {
      "firstName":{
        "type": "text",
        "copy_to": "fullName"
      },
      "lastName":{
        "type": "text",
        "copy_to": "fullName"
      }
    }
  }
}
# 增加一个文档
PUT users/_doc/1
{
  "firstName":"qing",
  "lastName": "zhuge"
}

# 尝试使用copy_to fullName进行search
GET users/_search?q=fullName:(zhuge qing)
```

### 设置默认的空值
```shell
DELETE users

PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "keyword",
          "null_value": "1111"
        }
      }
    }
}

PUT users/_doc/1
{
  "firstName": "qing",
  "lastName": "zhuge",
  "mobile" : null
}

GET users/_search
{
  "query": {
    "match": {
      "mobile": "1111"
    }
  }
}
```

### 数组类型
* Elasticsearch不提供数组类型，任意字段，都可以包含多个相同类型的数值

## 实战
```shell
# 增加文档
PUT mapping_test/_doc/1
{
  "firstName":"qing",
  "lastName": "zhuge",
  "loginDate":"2018-07-24"
}

# 查看索引的Mapping
GET mapping_test/_mapping

## 修改索引 mapping 的 dynamic
PUT dynamic_mapping_test/_mapping
{
  "dynamic": "false"
}
```