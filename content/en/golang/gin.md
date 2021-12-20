---
date: 2021-12-18
description: "对于Gin框架的初步使用"
image: "/images/Go.jpg"
title: "Go语言Gin框架基础"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 安装与示例

1. `go get -u github.com/gin-gonic/gin`来安装gin框架

2. 示例代码
```Go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 创建一个默认的路由引
	r := gin.Default()
	// GET：请求方式；/zhugeqing：请求的路径
	// 当客户端以GET方法请求/zhugeqing路径时，会执行后面的匿名函数
	r.GET("/zhugeqing", func(c *gin.Context) {
		// c.JSON：返回JSON格式的数据
		c.JSON(200, gin.H{
			"message": "Hello world!",
		})
	})
	// 启动HTTP服务，默认在8080端口进行监听
	r.Run()
}
```

## RESTful API
> Gin框架支持开发RESTful API的开发。
* 代码示例
```Go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default() //返回默认的路由
	r.GET("/zhugeqing", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "GET",
		})
	})

	r.POST("/zhugeqing", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "POST",
		})
	})

	r.PUT("/zhugeqing", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "PUT",
		})
	})

	r.DELETE("/zhugeqing", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "DELETE",
		})
	})
	//启动服务
	r.Run()
}
```

## Gin渲染


