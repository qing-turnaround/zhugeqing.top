---
date: 2021-10-09
description: "redis-go的学习"
image: "/images/Go.jpg"
title: "Go语言对Redis进行操作"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 选择框架
目前常用的两个Go语言操作redis的框架分别为[go-redis](https://github.com/go-redis/redis)和[redigo](https://github.com/gomodule/redigo)，推荐使用go-redis

## Installation
`go get github.com/go-redis/redis/v8`

查询使用文档[点击此处](https://pkg.go.dev/github.com/go-redis/redis/v8#section-readme)或者[点击此处](https://github.com/go-redis/redis#readme)

## 快速使用

```Go
package main

import (
	"context"
	"fmt"
	"github.com/go-redis/redis/v8"
)

//最新版的go-redis参数必须有context
var ctx = context.Background()

//初始化客户端连接
var rdb = redis.NewClient(&redis.Options{
	Addr:     "localhost:6379", //redis-serverip地址和端口号
	Password: "",               //密码设置
	DB:       0,                //选择数据库
	PoolSize: 100,              //连接池大小
})

func main() {

	//设置一个字符串类型的key，key为“key”，value为“value”，过期时间设置为永不过期
	err := rdb.Set(ctx, "key", "value", 0).Err()
	if err != nil {
		panic(err)
	}
	//查询"key"的值
	val, err := rdb.Get(ctx, "key").Result()
	if err != nil {
		panic(err)
	}
	//输出查询结果
	fmt.Println("key", val)

	val2, err := rdb.Get(ctx, "key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("key2", val2)
	}
	// 输出: key value
	// key2 does not exist
}
```

## set/get
```Go
func Example() {

	err := rdb.Set(ctx, "score", 100, 0).Err()
	if err != nil {
		fmt.Printf("set score failed, err:%v\n", err)
		return
	}

	val, err := rdb.Get(ctx, "score").Result()
	if err != nil {
		fmt.Printf("get score failed, err:%v\n", err)
		return
	}
	fmt.Println("score", val)

	val2, err := rdb.Get(ctx, "name").Result()
	if err == redis.Nil {
		fmt.Println("name does not exist")
	} else if err != nil {
		fmt.Printf("get name failed, err:%v\n", err)
		return
	} else {
		fmt.Println("name", val2)
	}
}
```

## zset
```Go
func Example2() {
	zsetKey := "language_rank"
	//当做可变参数传入
	languages := []*redis.Z{
		&redis.Z{Score: 90.0, Member: "Golang"},
		&redis.Z{Score: 98.0, Member: "Java"},
		&redis.Z{Score: 95.0, Member: "Python"},
		&redis.Z{Score: 97.0, Member: "JavaScript"},
		&redis.Z{Score: 99.0, Member: "C/C++"},
	}
	// ZADD
	num, err := rdb.ZAdd(ctx, zsetKey, languages...).Result()
	if err != nil {
		fmt.Printf("zadd failed, err:%v\n", err)
		return
	}
	fmt.Printf("zadd %d succ.\n", num)

	// 把Golang的分数加10
	newScore, err := rdb.ZIncrBy(ctx, zsetKey, 10.0, "Golang").Result()
	if err != nil {
		fmt.Printf("zincrby failed, err:%v\n", err)
		return
	}
	fmt.Printf("Golang's score is %f now.\n", newScore)

	// 取分数最高的3个
	ret, err := rdb.ZRevRangeWithScores(ctx, zsetKey, 0, 2).Result()
	if err != nil {
		fmt.Printf("zrevrange failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}

	// 取95~100分的
	op := redis.ZRangeBy{
		Min: "95",
		Max: "100",
	}
	ret, err = rdb.ZRangeByScoreWithScores(ctx, zsetKey, &op).Result()
	if err != nil {
		fmt.Printf("zrangebyscore failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}
}
```

## Sentinel
```Go
func Sentinel() error {
	SentinelRdb := redis.NewFailoverClient(&redis.FailoverOptions{
		MasterName: "master",
		//Sentinel密码
		SentinelPassword: "8888",
		//Sentinel地址和端口
		SentinelAddrs: []string{"x.x.x.x:26379", "xx.xx.xx.xx:26379", "xxx.xxx.xxx.xxx:26379"},
	})
	//产是否连接成功
	_, err := SentinelRdb.Ping(ctx).Result()
	if err != nil {
		return err
	}
	return nil
}
```

## Cluster
```Go
func Cluster() error {
	ClusterRdb := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},
	})
	_, err := ClusterRdb.Ping(ctx).Result()
	if err != nil {
		return err
	}
	return nil
}
```

## Pipeline

`Pipeline` 主要是一种网络优化。它本质上意味着客户端缓冲一堆命令并一次性将它们发送到服务器。这些命令不能保证在事务中执行。这样做的好处是节省了每个命令的网络往返时间（`RTT`）。

`Pipeline` 示例：
```Go
func Pipeline() {
	pipe := rdb.Pipeline()
	_, _ = pipe.Set(ctx, "诸葛青", "健康开心", 0).Result()
	_, _ = pipe.Set(ctx, "诸葛青1", "无限可能", 0).Result()
	_, _ = pipe.Incr(ctx, "22").Result()
	//执行Pipeline里面的所有命令
	_, _ = pipe.Exec(ctx)
}
```

## 事务

Redis是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是，`Multi/exec`能够确保在`multi/exec`两个语句之间的命令之间没有其他客户端正在执行命令。

在这种场景我们需要使用`TxPipeline`。`TxPipeline`总体上类似于上面的`Pipeline`，但是它内部会使用`MULTI/EXEC`包裹排队的命令。

`transaction` 示例：

```Go
func transaction() {
	tx := rdb.TxPipeline()
	_ = tx.Set(ctx, "number 1",  "One", 0)
	_ = tx.Set(ctx, "number 2", "two", 0)
	_ = tx.Incr(ctx, "22")
	_, _ = tx.Exec(ctx)
}
```

## Watch

在某些场景下，除了要使用`MULTI/EXEC`命令外，还需要配合使用`WATCH`命令。在用户使用`WATCH`命令监视某个键之后，直到该用户执行`EXEC`命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行`EXEC`的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。

`Watch`示例
```Go
func Watch() {
	// 监视watch_count的值，并在值不变的前提下将其值+1
	key := "watch_count"
	err := rdb.Watch(ctx,func(tx *redis.Tx) error {
		n, err := tx.Get(ctx,key).Int()
		if err != nil && err != redis.Nil {
			return err
		}
		_, err = tx.Pipelined(ctx,func(pipe redis.Pipeliner) error {
			pipe.Set(ctx,key, n+1, 0)
			return nil
		})
		return err
	}, key)
	if err != nil {
		fmt.Printf("error: %v\n", err)
	}
	fmt.Println("Exec succeeded")
}
```
