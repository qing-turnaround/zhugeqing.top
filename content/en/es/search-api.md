---
date: 2022-09-18
description: "elasticsearch search api使用"
image: "images/es.png"
title: "elasticsearch search api使用"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- elasticsearch
series:
- 
---

## URL Search（通过URL query进行搜索）

### 基本了解
```shell
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10
{
  "profile": "true"
}
```
* q 指定查询语句，后面接Query String Syntax
* df指定查询的字段，不指定时，会对所有字段进行查询
* Sort 排序，form 和 size用于分页
* profile 查看查询是如何被执行的

### 指定字段和泛查询
* 指定字段：q=title:2012
* 泛查询：q=2012
* Trem查询：q=(Happy Mind) （等效于Happy or Mind）
* Phrase查询：q="Happy Mind"（等效于Happy and Mind，并且要求顺序保持一致）
* 布尔操作
    * AND (&&)
    * OR  (||)
    * NOT (!)
* 分组
    * + 代表Must
    * - 代表Must_not
    * 例子：q=title:(-Beautiful +Mind)

* 区间查询
    * []是闭区间，{}是开区间
    * year:{2022 TO 2024}   
    * year:[* TO 2023]
 
* 通配符查询（占用内存大，不建议使用）
    * ?代表一个字符，*代表0或多个字符
    * title:beautifu?
    * title:be*
* 模糊匹配
    * q=title:balck~2（解释：允许balck增删改2个字符得到最终匹配的单词）

* 近似度匹配
    * q=title:"Happy Mind"~2 （解释：允许Happy 和 Mind中间相隔N个单词）

## RequestBody
* 将查询语句 通过 HTTP Request Body发送给 Elasticsearch


### Term Query
```shell
POST usres/_search
{
  "query": {
    "term": {
      "name": "诸葛青"
    }
  }
}
```


### match Query
```shell
POST movies/_search
{
  "sort": [
    {
      "year": {
        "order": "desc"
      }
    }
  ],
  
  "query": {
    "match": {
      "title": {
        "query": "Beautiful Mind",
        "operator": "AND"
      }
    }
  }
  
}
```

### match_pharse Query
```shell
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "Beautiful A",
        "slop": "3"
      }
    }
    }
}
```

### query_string
```shell
POST users/_search
{
  "query": {
    "query_string": {
      "default_field": "name",
      "query": "Ruan AND Yiming"
    }
  }
}
```
> 类似URL query 

## 学习感受
* 进一步学习
> https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html
* Elasticsearch提供了相当多查询，无法全部记住，只能学习常用的部分