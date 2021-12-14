---
date: 2021-11-11
description: "现在的我不过是无数个“可能的我”中的一个样本"
image: "/images/Go.jpg"
title: "Go语言http详解"
author: Go web 编程
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 基础概念
1. Request
> Request是http请求，是用户请求的信息，用来解析用户的请求信息，包括post、get、cookie、url等信息，可以访问[源码地址](https://go.dev/src/net/http/request.go)（源码103行）来查看Request结构体的详细定义

2. Response
> Response是http请求的响应，是服务器需要反馈给客户端的信息，可以访问[源码地址](https://go.dev/src/net/http/response.go)（源码35行）来查看Response结构体的详细定义

3. Conn
> Conn是http的请求连接（用户的每次请求连接），可以访问[源码地址](https://go.dev/src/net/http/server.go)（源码248行）来查看Response结构体的详细定义    

4. Handler
> Handler是接收请求后逻辑处理和生成返回信息的逻辑，可以访问[源码地址](https://go.dev/src/net/http/server.go)（源码86行）来查看Handler接口

## http包运行机制
`Go实现Web服务的工作模式的流程图`
![http运行图](/images/golang/http_run.png)

### http包执行流程😡
1. `创建Listen Socket, 监听指定的端口, 等待客户端请求到来。`

2. `Listen Socket接受客户端的请求, 得到Client Socket, 接下来通过Client Socket与客户端通信。`

3. `处理客户端的请求, 首先从Client Socket读取HTTP请求的协议头, 如果是POST方法, 还可能要读取客户端提交的数据, 然后交给相应的handler处理请求, handler处理完毕准备好客户端需要的数据, 通过Client Socket写给客户端。`

### 三个应该清楚的问题😃

1. `如何监听端口？`

Go是通过一个函数`ListenAndServe`来处理这些事情的，这个底层其实这样处理的：初始化一个server对象，然后调用了`net.Listen("tcp", addr)`，也就是底层用TCP协议搭建了一个服务，然后监控我们设置的端口。

2. `如何接收客户端请求？`

执行监控端口之后，调用了``srv.Serve(net.Listener)``函数（代码如下），这个函数就是``处理接收客户端的请求信息``。这个函数里面起了一个``for{}``，首先通过``Listener``接收请求，其次创建一个``Conn``，最后单独开了一个``goroutine``，把这个请求的数据当做参数扔给这个conn去服务：``go c.serve()``。用户的每一次请求都是在一个新的``goroutine``去服务，相互不影响
```Go
func (srv *Server) Serve(l net.Listener) error {
    defer l.Close()
    var tempDelay time.Duration // how long to sleep on accept failure
    for {
        rw, e := l.Accept()
        if e != nil {
            if ne, ok := e.(net.Error); ok && ne.Temporary() {
                if tempDelay == 0 {
                    tempDelay = 5 * time.Millisecond
                } else {
                    tempDelay *= 2
                }
                if max := 1 * time.Second; tempDelay > max {
                    tempDelay = max
                }
                log.Printf("http: Accept error: %v; retrying in %v", e, tempDelay)
                time.Sleep(tempDelay)
                continue
            }
            return e
        }
        tempDelay = 0
        c, err := srv.newConn(rw)
        if err != nil {
            continue
        }
        go c.serve() //创建一个Goroutine来处理
    }
}
```

3. 如何分配handler？

`conn`首先会解析`request:c.readRequest()`,然后获取相应的`handler:handler := c.server.Handler`，也就是在调用函数`ListenAndServe`时候的第二个参数，如果传递的是`nil`，也就是为空，那么默认获取`handler = DefaultServeMux`,这个变量是一个路由器，它用来匹配`url`跳转到其相应的`handle`函数，而`http.HandleFunc("/", handle)`就是注册了请求`/`的路由规则，当请求uri为"\"，路由就会转到函数`handle`，`DefaultServeMux`会调用`ServeHTTP`方法，这个方法内部其实就是调用`handle`本身，最后通过写入`response`的信息反馈到客户端

`一个http连接处理流程`
![http执行流程图](/images/golang/http_run1.png)


## 代码分析

### 代码
```Go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.Handle("/", &ThisHandler{})
	http.Handle("/彩虹海", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("恭喜你已经成抵达彩虹海！"))
	}))
	http.HandleFunc("/诸葛青", sayHi)
	http.ListenAndServe(":8080", nil)
}

func sayHi(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hi,诸葛青,%v", "恭喜你已经初步掌握Go http包的基本工作原理了")
}

type ThisHandler struct{}

func (m *ThisHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("我实现了Hanler这个接口"))
}
```
### 分析
1. 第9行的`http.Handle("/", &ThisHandler{})`表示为请求路径`/`注册了一个默认的路由器，而且`ThisHanler`这个struct实现了`ServeHTTP`方法，所以也就实现了接口Handler，固可以传入到函数`http.Handle`中。`http.Handle函数`的源码如下：
```Go
func Handle(pattern string, handler Handler) { 
    DefaultServeMux.Handle(pattern, handler) 
}
```

2. 第10~12行同样调用了`函数http.Handle`，但所传入参数与上面调用的不同，上面的第二个参数是因为这个结构体实现了`Handle`接口的方法，所以可以传入，而这里的`http.HandlerFunc`实际是`type HandlerFunc func(ResponseWriter, *Request)`，而在源码中这个类型别名同样实现了`ServeHTTP(w ResponseWriter, r *Request)`这个方法，所以这里相当于是一个类型转换，将实际的处理函数转换成`http.HandleFunc`类型，所以也可以传入到`http.Handle`中。具体源码如下：
```GO
//HandlerFunc类型是一个适配器，允许使用作为 HTTP处理程序的普通函数
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP 调用这个函数自身
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

3. 第13行调用了`http.HandleFunc`这个函数，功能与`http.Handle`差不多。只不过`http.Handle`的第二个参数是一个Handler接口类型，也就是说只要实现了这个接口的类型都能作为参数传入，而`http.HandleFunc`的第二个参数就是一个函数类型，只要传入的函数以`func(http.ResponseWriter, *http.Request)`形式声明即可。具体源码如下：
```Go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

4. 第14行调用了`http.ListenAndServe`函数，第一个参数为监听的address，第二个参数为Handler接口类型，因为传入的是nil，所以handler就是默认的`DefaultServeMux`，也就是上面调用的函数源码中的`DefaultServeMux`。部分源码如下：
```GO
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

## http包核心
Go的http有两个核心功能：`Conn`、`ServeMux`

### Conn
Go使用了goroutines来处理Conn的读写事件, 这样每个请求都能保持独立，相互不会阻塞，可以高效的响应网络事件。
Go在等待客户端请求的源码如下：
```Go
c, err := srv.newConn(rw)
if err != nil {
    continue
}
go c.serve()
```

### ServeMux
> 相当于是http内部的路由器

`ServeMux`的结构如下

```Go
type ServeMux struct {
	mu    sync.RWMutex //读写锁
	m     map[string]muxEntry // 路由 map，pattern->HandleFunc
	es    []muxEntry // 从长到短排序好的的切片
	hosts bool       // 是否包含hosts
}

type muxEntry struct {
	h       Handler //路由对应的Handler
	pattern string  //路由路径
}
```

在ServeMux中的成员m就是关于不同的路由路径所对应的处理函数的映射，如果找不到对于的路径映射，那么就是默认返回`404`，但是如果配置了`/`的处理函数，那么对于没有对应路由路径的会调用`/`的处理函数

## 总结

* 首先调用`h

ttp.Handle`和`http.HandleFunc`
> 按顺序做了几件事
>1.调用了`DefaultServeMux的Handle`
>2.调用了`DefaultServeMux的HandleFunc`
>3.往`DefaultServeMux`的`map[string]muxEntry`中增加对应的`handler和路由规则`

<font color=SkyBlue>  </font>

* 其次调用`http.ListenAndServe(":8080", nil)`
>按顺序做了几件事情
>1.实例化`Server`
>2.调用`Server`的`ListenAndServe()`
>3.调用`net.Listen("tcp", addr)`监听端口
>4.启动一个`for循环`，在循环体中`Accept`请求
>5.对每个请求实例化一个`Conn`，并且开启一个`goroutine`为这个请求进行服务`go c.serve()`
>6.读取每个请求的内容`w, err := c.readRequest()`
>7.根据`request`选择`handler`，并且进入到这个`handler`的`ServeHTTP`
>8.判断`handler`是否为空，如果没有设置`handler`，handler就设置为`DefaultServeMux`
>9.选择`handler`：
A 判断是否有路由能满足这个`request`（循环遍历`ServeMux`的`muxEntry`）
B 如果有路由满足，调用这个路由`handler`的`ServeHTTP`
C 如果没有路由满足，调用`NotFoundHandler`的`ServeHTTP`
