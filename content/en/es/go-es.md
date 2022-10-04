---
date: 2022-09-18
description: "go 语言操作Elasticsearch"
image: "images/es.png"
title: "go 语言操作Elasticsearch"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- elasticsearch
series:
- 
---
> 参考：https://olivere.github.io/elastic/
## 初始化客户端
```go:main.go
package main

import (
	"context"
	"fmt"
	"github.com/olivere/elastic/v7"
	"log"
)

var esUrl = "http://127.0.0.1:9200/" // es 地址

func main() {
	client, err := elastic.NewClient(elastic.SetSniff(false), elastic.SetURL(esUrl))
	if err != nil {
		log.Fatalln(err)
	}
	ctx := context.Background()
	result, code, err := client.Ping(esUrl).Do(ctx)
	fmt.Println(result, code, err)

}
```

## document_api

### Post一个文档
```go
func Post(client *elastic.Client) {
	user := User{Age: 20, Name: "zhugeqing"}

	// 不指定ID
	//res, err := client.Index().Index(user.GetIndexName()).BodyJson(user).Do(context.Background())

	// 指定ID
	res, err := client.Index().Index(user.GetIndexName()).Id("100").BodyJson(user).Do(context.Background())
	if err != nil {
		panic(err)
	}
	// 输出文档ID
	fmt.Println(res.Id)
}

type User struct {
	Age int `json:"age"`
	Name string `json:"name"`
}

// GetIndexName: 获取索引名
func (u User) GetIndexName() string {
	return "users"
}
```

### Get一个文档
```go
func Get(client *elastic.Client) {
	res, err := client.Get().Index("users").Id("100").Do(context.Background())
	if err != nil {
		panic(err)
	}
	source, _ := res.Source.MarshalJSON()
	fmt.Println(string(source))
}
```

### Delete一个文档
```go
func Delete(client *elastic.Client) {
	deleteRes, err := client.Delete().Index(User{}.GetIndexName()).Id("100").Do(context.Background())
	if err != nil {
		panic(err)
	}
	fmt.Println(deleteRes.Result)
}
```


## match查询
> https://github.com/olivere/elastic/blob/release-branch.v7/search_queries_match_test.go
```go
func Match(client *elastic.Client) {
	// 去 索引 users 中查询 name是zhugeqing的

	indexName := "users"
	query := elastic.NewMatchQuery("name", "zhugeqing")

	res, err := client.Search().Index(indexName).Query(query).Do(context.Background())
	if err != nil {
		panic(err)
	}
	fmt.Println(res.Hits.TotalHits.Value) // 查看搜索数量
	for _, v := range res.Hits.Hits {    // 查看搜索结果的hit
		fmt.Println(v.Id) // 查看文档Id

		// 查看文档Source
		if json, err := v.Source.MarshalJSON(); err == nil {
			fmt.Println(string(json))
		} else {
			panic(err)
		}
	}
}
```
## 将Elasticsearch 文档数据转换为 go 结构体
```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"github.com/olivere/elastic/v7"
)

type User struct {
	Age int `json:"age"`
	Name string `json:"name"`
}

func ConvertToStruct(client *elastic.Client) {
	// 去 索引 users 中查询 name是zhugeqing的

	indexName := "users"
	query := elastic.NewMatchQuery("name", "zhugeqing")

	res, err := client.Search().Index(indexName).Query(query).Do(context.Background())
	if err != nil {
		panic(err)
	}
	fmt.Println(res.Hits.TotalHits.Value) // 查看搜索数量
	for _, v := range res.Hits.Hits {    // 查看搜索结果的hit
		user := &User{}
		// 解析到结构体中
		if err := json.Unmarshal(v.Source, user); err != nil {
			panic(err)
		}

		fmt.Println("the user is", user)
	}
}
```

## 追踪es请求的发送
```go
func TraceElastic() {
	logger := log.New(os.Stdout, "es-trace:", log.LstdFlags)

	client, err := elastic.NewClient(elastic.SetURL(esUrl), elastic.SetSniff(false),
		elastic.SetTraceLog(logger))
	if err != nil {
		panic(err)
	}
	// 保存数据到es中
	Update(client)
}
```

## 使用go语言 向es创建Mapping
```go
// 商品索引
type Goods struct {
	Id int `json:"id"`
	Name string `json:"name"`
}

func (Goods)getIndexName() string {
	return "goods"
}

const mapping = `
{
	"settings":{
		"number_of_shards": 1,
		"number_of_replicas": 0
	},
	"mappings":{
		"properties":{
			"name":{
				"type":"text",
				"analyzer":"ik_max_word"
			},
			"id":{
				"type":"integer"
			}
		}
	}
}
`

func CreateMapping(client *elastic.Client) {
	createIndexResult, err := client.CreateIndex("goods").BodyString(mapping).Do(context.Background())
	if err != nil {
		panic(err)
	}

	fmt.Println(createIndexResult.Acknowledged)
}
```

## 遇到的坑

### 无法连通docker中部署的elasticsearch

* 使用olivere/elastic时，连接es会将输入的网址转换成内网地址或者是docker中的IP地址，导致无法连接
* 可以设置elastic.SetSniff(false)
