---
date: 2022-06-01
description: "Go 语言操作rocketmq"
image: "/images/Go.jpg"
title: "Go 语言操作rocketmq"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 普通消息

### 生产者
> https://github.com/apache/rocketmq-client-go/blob/master/examples/producer/async/main.go
```go:producer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"github.com/apache/rocketmq-client-go/v2/producer"
	"time"
)


func main() {
	p, err := rocketmq.NewProducer(
		producer.WithNameServer([]string{"192.168.200.120:9876"}),
		producer.WithGroupName("producer_group1"),
		producer.WithRetry(3), // 失败重试次数
	)
	if err != nil {
		panic(err)
	}

	p.Start() // 启动producer

	for i := 0; i < 10; i++ {
		// 发送消息
		err = p.SendAsync(context.Background(), // 异步发送
			func(ctx context.Context, result *primitive.SendResult, err error) {
				if err != nil {
					fmt.Println("rocketmq send error:", err)
				} else {
					fmt.Println("rocketmq send success, result:", result)
				}

			}, &primitive.Message{
				Topic: "topic1",
				Body:  []byte("I am producer"),
			})
		if err != nil {
			panic(err)
		}
	}

	time.Sleep(time.Second * 40)
	p.Shutdown() // 关闭producer
}
```

### 消费者
> https://github.com/apache/rocketmq-client-go/blob/master/examples/consumer/acl/main.go
```go:consumer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/consumer"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"sync"
	"time"
)

var (
	a int
	mu sync.Mutex
)


func main() {
	c, err := rocketmq.NewPushConsumer(
		consumer.WithNameServer([]string{"192.168.200.120:9876"}),
		consumer.WithGroupName("consumer_group1"),
	)
	if err != nil {
		panic(err)
	}
	err = c.Subscribe("topic1", consumer.MessageSelector{},
		func(ctx context.Context, msgs ...*primitive.MessageExt) (consumer.ConsumeResult, error) {
			for i := range msgs {
				fmt.Println("消费内容：", msgs[i].Body)
			}
			add()
			return consumer.ConsumeSuccess, nil // 返回消费结果
		})
	if err != nil {
		panic(err)
	}

	c.Start() // 启动消费者

	time.Sleep(time.Second * 20)
	if err := c.Shutdown(); err != nil {
		panic(err)
	}

	fmt.Println(a) // 查看是否 消费了10次
}

func add() {
	mu.Lock()
	a++
	mu.Unlock()
}
```

## 顺序消息

### 生产者
```go:producer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"github.com/apache/rocketmq-client-go/v2/producer"
	"time"
)


type QueueSelector struct {
}

func (QueueSelector) Select(msg *primitive.Message, queue []*primitive.MessageQueue) *primitive.MessageQueue {
	return queue[0] // 固定选择
}

// 顺序发送
func main() {
	p, err := rocketmq.NewProducer(
		producer.WithNameServer([]string{"42.192.19.149:9876"}),
		producer.WithGroupName("producer_group2"),
		producer.WithRetry(3), // 失败重试次数
		producer.WithQueueSelector(QueueSelector{}), // 只发送到一个queue中
	)
	if err != nil {
		panic(err)
	}

	p.Start() // 启动producer

	for i := 0; i < 10; i++ {
		// 发送消息
		result, err := p.SendSync(context.Background(), // 使用同步发送
			&primitive.Message{
				Topic: "topic2",
				Body: []byte(fmt.Sprintf("I am producer, send %d times, test", i+1)),
			})
		if err != nil {
			panic(err)
		} else {
			fmt.Println("producer send result is ", result)
		}
	}

	time.Sleep(time.Second * 10)
	p.Shutdown() // 关闭producer
}
```

### 消费者
```go:consumer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/consumer"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"sync"
	"time"
)

var (
	mu sync.Mutex
)

// FIFO 消费
func main() {
	c, err := rocketmq.NewPushConsumer(
		consumer.WithNameServer([]string{"42.192.19.149:9876"}),
		consumer.WithGroupName("consumer_group2"),
		consumer.WithConsumerModel(consumer.Clustering),
		consumer.WithConsumeFromWhere(consumer.ConsumeFromFirstOffset),
		consumer.WithConsumerOrder(true),
	)
	if err != nil {
		panic(err)
	}
	err = c.Subscribe("topic2", consumer.MessageSelector{},
		func(ctx context.Context, msgs ...*primitive.MessageExt) (consumer.ConsumeResult, error) {
			// 调用函数进行消费
			Consumer(msgs)
			return consumer.ConsumeSuccess, nil // 返回消费结果
		})
	if err != nil {
		panic(err)
	}

	c.Start() // 启动消费者

	time.Sleep(time.Second * 50)
	if err := c.Shutdown(); err != nil {
		panic(err)

	}
}

func Consumer(msgs []*primitive.MessageExt) {
	mu.Lock()
	for i := range msgs {
		fmt.Println("消费内容：", string(msgs[i].Body))
	}
	mu.Unlock()
}
```

## 延时消息

### 生产者
```go:producer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"github.com/apache/rocketmq-client-go/v2/producer"
	"time"
)


type QueueSelector struct {
}


// 延时发送
func main() {
	p, err := rocketmq.NewProducer(
		producer.WithNameServer([]string{"42.192.19.149:9876"}),
		producer.WithGroupName("producer_group3"),
		producer.WithRetry(3), // 失败重试次数
	)
	if err != nil {
		panic(err)
	}

	p.Start() // 启动producer

	for i := 0; i < 10; i++ {
		// 发送消息
		msg := 	&primitive.Message{
			Topic: "topic3",
			Body: []byte(fmt.Sprintf("I am producer, send %d times, test", i+1)),
		}
		msg.WithDelayTimeLevel(4) // 设置延时发送级别 第4个级别是30s
		result, err := p.SendSync(context.Background(), msg)

		if err != nil {
			panic(err)
		} else {
			fmt.Println("producer send result is ", result)
		}
	}

	time.Sleep(time.Second * 50)
	p.Shutdown() // 关闭producer
}
```

### 消费者
```go:consumer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/consumer"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"sync"
	"time"
)

var (
	mu sync.Mutex
)

// FIFO 消费
func main() {
	c, err := rocketmq.NewPushConsumer(
		consumer.WithNameServer([]string{"42.192.19.149:9876"}),
		consumer.WithGroupName("consumer_group2"),
	)
	if err != nil {
		panic(err)
	}
	err = c.Subscribe("topic3", consumer.MessageSelector{},
		func(ctx context.Context, msgs ...*primitive.MessageExt) (consumer.ConsumeResult, error) {
			// 调用函数进行消费
			Consumer(msgs)
			return consumer.ConsumeSuccess, nil // 返回消费结果
		})
	if err != nil {
		panic(err)
	}

	c.Start() // 启动消费者

	time.Sleep(time.Second * 50)
	if err := c.Shutdown(); err != nil {
		panic(err)

	}
}

func Consumer(msgs []*primitive.MessageExt) {
	mu.Lock()
	for i := range msgs {
		fmt.Println("消费内容：", string(msgs[i].Body))
	}
	mu.Unlock()
}
```

## 事务消息

### 生产者
```go:producer.go
package main

import (
	"context"
	"errors"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"github.com/apache/rocketmq-client-go/v2/producer"
	uuid "github.com/satori/go.uuid"
	"time"
)

// 实现事务消息 进行回调的接口
type Listener struct {

}

func (*Listener)ExecuteLocalTransaction(*primitive.Message) primitive.LocalTransactionState {
	// 执行本地事务逻辑
	// 进行库存扣减
	var err = errors.New("本地事务执行失败")
	if err != nil {
		return primitive.CommitMessageState // 执行失败，需要通知库存服务归还库存，将半消息发送给 库存服务
	}

	return primitive.RollbackMessageState // 执行成功，无须再归还库存了，丢弃半消息
}

func (*Listener)CheckLocalTransaction(*primitive.MessageExt) primitive.LocalTransactionState {
	// 回查就会调用这个函数，这个函数执行检查
	var err = errors.New("本地事务执行失败")
	if err != nil { // 回查 发现本地事务依旧失败
		return primitive.CommitMessageState
	}

	return primitive.RollbackMessageState
}

// 事务消息
func main() {
	p, err := rocketmq.NewTransactionProducer(&Listener{},
		producer.WithNameServer([]string{"42.192.19.149:9876"}),
		producer.WithGroupName("producer_group4"),
		producer.WithRetry(3), // 失败重试次数
	)
	if err != nil {
		panic(err)
	}

	p.Start() // 启动producer

	// 发送半消息
	p.SendMessageInTransaction(context.Background(), &primitive.Message{
		Topic:          "reback", // 归还商品库存
		Body:          []byte("please reback"),
		TransactionId:  uuid.NewV4().String(),
	})

	time.Sleep(time.Second * 200)
	p.Shutdown() // 关闭producer
}
```

### 消费者
```go:consumer.go
package main

import (
	"context"
	"fmt"
	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/consumer"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"sync"
	"time"
)

var (
	mu sync.Mutex
)

func main() {
	c, err := rocketmq.NewPushConsumer(
		consumer.WithNameServer([]string{"42.192.19.149:9876"}),
		consumer.WithGroupName("consumer_group4"),
		consumer.WithConsumerModel(consumer.Clustering),
		consumer.WithConsumeFromWhere(consumer.ConsumeFromFirstOffset),
		consumer.WithConsumerOrder(true),
	)
	if err != nil {
		panic(err)
	}
	err = c.Subscribe("reback", consumer.MessageSelector{},
		func(ctx context.Context, msgs ...*primitive.MessageExt) (consumer.ConsumeResult, error) {
			// 调用函数进行消费
			Consumer(msgs)
			return consumer.ConsumeSuccess, nil // 返回消费结果
		})
	if err != nil {
		panic(err)
	}

	c.Start() // 启动消费者

	time.Sleep(time.Second * 50)
	if err := c.Shutdown(); err != nil {
		panic(err)

	}
}

func Consumer(msgs []*primitive.MessageExt) {
	mu.Lock()
	for i := range msgs {
		fmt.Println("消费内容：", string(msgs[i].Body))
	}
	mu.Unlock()
}
```