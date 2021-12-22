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


### HTML渲染

存放模板文件文件夹：`templates`，再在其内部分别定义一个`posts`文件夹和一个`users`文件夹。 
* 模版文件`posts/index.tmpl`
```HTML
{{define "posts/index.tmpl"}}
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>posts/index</title>
</head>
<body>
    {{.title}}
</body>
</html>
{{end}}
```

* 模版文件`users/index.tmpl`
```HTML
{{define "users/index.tmpl"}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>users/index</title>
</head>
<body>
    {{.title}}
</body>
</html>
{{end}}
```

`Gin`框架中使用`LoadHTMLGlob()`或者`LoadHTMLFiles()`方法进行`HTML模板解析`

* 代码文件`main.go`
```Go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()               //获取一个gin引擎
	r.LoadHTMLGlob("templates/**/*") //解析模版：正则匹配传入模版文件位置

	r.GET("/posts/index", func(c *gin.Context) {
		//渲染模版
		c.HTML(http.StatusOK, "posts/index.tmpl", gin.H{ //name就是模版文件定义的名字
			"title": "posts/index",
		})
	})

	r.GET("users/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "users/index.tmpl", gin.H{
			"title": "users/index",
		})
	})

	r.Run(":8080")
}
```

### 自定义模板函数
> 自定义函数需要在解析模版文件之前设置好

* 代码文件`main.go`
```Go
package main

import (
	"github.com/gin-gonic/gin"
	"html/template"
	"net/http"
)

func main() {
	r := gin.Default() //获取一个gin引擎
	//gin框架 自定义模版函数
	r.SetFuncMap(template.FuncMap{
		"safe": func(s string) template.HTML {
			return template.HTML(s)
		},
	})
	r.LoadHTMLFiles("go.tmpl") //解析模版：正则匹配传入模版文件位置
	r.GET("/", func(c *gin.Context) {
		//渲染模版
		c.HTML(http.StatusOK, "go.tmpl", "<a href=\"https://zhugeqing.top\">诸葛青的编程之旅</a>")
	})

	r.Run(":8080")
}
```

* 模版文件`go.tmpl`
```HTML
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>gin 自定义模版引擎</title>
</head>
<body>
<div>{{ . | safe }}</div>
</body>
</html>
```

### 静态文件处理

>当渲染的HTML文件中引用了`静态文件`时，在渲染页面前调用`gin.Static`方法
```Go
func main() {
	r := gin.Default()
	//第一个参数的名字需要和在HTML文件中引用的css文件的路径一样
	r.Static("/static", "./static")
s	r.LoadHTMLGlob("templates/**/*")
   // ...
	r.Run(":8080")
}
```

### JSON渲染

```Go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()

	r.GET("/jsonMap", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"name":    "诸葛青",
			"message": "你好！",
			"age":     19})
	})

	r.GET("/jsonStruct", func(c *gin.Context) {
		c.JSON(http.StatusOK, struct {
			Name    string `json:"名字"`
			Message string `json:"信息"`
			Age     int    `json:"年龄"`
		}{"诸葛青", "你好！", 19})
	})

	r.Run(":8080")
}
```

## 参数获取

### 获取queryString参数

>`querystring`指的是URL中`?`后面携带的参数，例如：`/web?country=China&city=北京`

```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()

	r.GET("/web", func(c *gin.Context) {
		//查询参数为country和city，若在网址中有指定值，则返回对应结果
		country := c.DefaultQuery("country", "China")
		city := c.Query("city")

		if city == "" {
			city = "广州"
		}
		if country == "China" {
			c.JSON(http.StatusOK, gin.H{
				"城市": city + "是一个美丽的城市",
				"国家": fmt.Sprintf("%v是一个强大的国家", country),
			})
		} else {
			c.JSON(http.StatusOK, gin.H{
				"城市": city + "是一个美丽的城市",
				"国家": fmt.Sprintf("%v是一个还行的国家", country),
			})
		}
	})

	r.Run(":8080")
}
```

### 获取form参数
> 当请求的数据通过form表单提交时，例如向/login发送一个POST请求，获取请求数据的方式如下：

```Go
func main() {
	//Default返回一个默认的路由引擎
	r := gin.Default()
	//POST请求来提交表单
	r.POST("/login", func(c *gin.Context) {
		// DefaultPostForm取不到值时会返回指定的默认值
		//username := c.DefaultPostForm("username", "********")
		username := c.PostForm("username")
		password := c.PostForm("password")
		//输出json结果给调用方
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  password,
		})
	})
	r.Run(":8080")
}
```

### 获取path参数

> 请求的参数通过`URL`路径传递，例如：`/user/search/诸葛青/星游记`。 获取请求URL路径中的参数的方式如下。
```Go
func main() {
	//Default返回一个默认的路由引擎
	r := gin.Default()
	r.GET("/user/search/:username/:favorite", func(c *gin.Context) {
		//获取路径参数
		username := c.Param("username")
		favorite := c.Param("favorite")
		//输出json结果给调用方
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  favorite,
		})
	})

	r.Run(":8080")
}
```

### 参数绑定

> 基于请求的`Content-Type`识别请求数据类型并利用`反射机制自`动提取请求中`QueryString`、`form表单`、`JSON`、`XML`等参数到结构体中。 `ShouldBind()`示例：
* 如果是 GET 请求，只使用 Form 绑定引擎（query）。
* 如果是 POST 请求，首先检查 content-type 是否为 JSON 或 XML，然后再使用 Form（form-data）。

```Go
// Binding from JSON
type Login struct {
	User     string `form:"user" json:"user" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}

func main() {
	router := gin.Default()

	// 绑定JSON的示例 ({"user": "root", "password": "123456"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var login Login

		if err := c.ShouldBind(&login); err == nil {
			fmt.Printf("login info:%#v\n", login)
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	// 绑定form表单示例 (user=root&password=123456)
	router.POST("/loginForm", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	// 绑定QueryString示例 (/loginQuery?user=root&password=123456)
	router.GET("/loginQuery", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	router.Run(":8080")
}
```

