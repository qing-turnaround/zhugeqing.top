---
date: 2022-03-03
description: "Go实战微服务网关"
image: "/images/Go.jpg"
title: "Go 微服务网关"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 网络代理

* `网络代理`：`用户通过代理发送请求消息`；`请求通过网络代理转发到目标服务器`；`目标服务器的响应通过网络代理回传给用户`

* `网络代理`与`网络转发`的区别：
    * `网络代理`：用户不直接连接服务器，由网络代理去连接，获取数据后返回给用户。
    * `网络转发`：是路由器对报文的转发操作，中间可能对数据包修改


### 正向代理
> 一种`客户端`的代理技术，帮助客户端访问无法访问的服务资源，可以隐藏用户真实IP。比如：浏览器web代理，VPN等


`正向代理示例：（直接运行，同时需要打开电脑的代理设置，并设置与程序同步的Host:Port）`
* 代理接受客户端请求，复制原请求对象，并根据数据配置新请求各种参数
* 把新请求发送到真实服务端，并接受到服务端的返回
* 代理服务器对响应做一些处理，然后返回给客户端
```Go
package main

import (
	"fmt"
	"io"
	"net"
	"net/http"
	"strings"
)

type Pxy struct{}

func (p *Pxy) ServeHTTP(rw http.ResponseWriter, req *http.Request) {
	// 打印请求的对应字段
	fmt.Printf("Received request %v %v %v\n", req.Method, req.Host, req.RemoteAddr)
	transport := http.DefaultTransport // 定义一个数据连接池
	// step 1，浅拷贝
	outReq := new(http.Request)
	*outReq = *req
	if clientIP, _, err := net.SplitHostPort(outReq.RemoteAddr); err != nil {
		fmt.Println("clientIP is", clientIP)
		if v, ok := outReq.Header["X-Forwarded-For"]; ok {
			clientIP = strings.Join(v, ",") + ", " + clientIP
		}
		outReq.Header.Set("X-Forwarded-For", clientIP)
	}

	// step 2, 请求下游
	response, err := transport.RoundTrip(outReq)
	if err != nil {
		rw.WriteHeader(http.StatusBadGateway)
		return
	}

	// step3, 将下游的请求内容返回给上游
	for key, value := range response.Header {
		for _, v := range value {
			rw.Header().Add(key, v)
		}
	}
	rw.WriteHeader(response.StatusCode) // 正常响应
	io.Copy(rw, response.Body)
	response.Body.Close()
}

func main() {
	fmt.Println("Listening on :8080")
	http.Handle("/", &Pxy{})
	http.ListenAndServe("0.0.0.0:8080", nil)
}
```

### 反向代理
> 一种`服务端`的代理技术，帮助服务器做`负载均衡`、`缓存`、`安全校验`等，可以隐藏服务器真实IP，比如：Nginx Proxy_pass等

`反向代理示例：（运行两个程序，一个服务端程序，一个反向代理程序）`
* 代理接受客户端请求，更改请求结构体信息
* 通过一定的负载均衡算法获取下游服务地址
* 把请求发送到下游服务器，并返回响应
`服务器代码`
```Go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

type RealServer struct {
	Addr string
}

func (r *RealServer) HelloHandler(w http.ResponseWriter, req *http.Request) {
	uPath := fmt.Sprintf("http://%s%s\n", r.Addr, req.URL.Path)
	io.WriteString(w, uPath)
}

func (r *RealServer) ErrorHandler(w http.ResponseWriter, req *http.Request) {
	uPath := "error handler"
	w.WriteHeader(http.StatusInternalServerError)
	io.WriteString(w, uPath)
}

func (r *RealServer) Run() {
	log.Println("Starting http server at" + r.Addr)
	mux := http.NewServeMux()
	mux.HandleFunc("/", r.HelloHandler)
	mux.HandleFunc("/base/error", r.ErrorHandler)
	server := &http.Server{
		Addr:         r.Addr,
		WriteTimeout: time.Second * 3,
		Handler:      mux,
	}
	go func() {
		log.Fatal(server.ListenAndServe())
	}()
}

func main() {
	rs1 := &RealServer{"127.0.0.1:8081"}
	rs2 := &RealServer{"127.0.0.1:8082"}
	rs1.Run()
	rs2.Run()

	// 监听关闭信号
	quit := make(chan os.Signal)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit
}
```

`反向代理代码`
```Go
package main

import (
	"bufio"
	"log"
	"net/http"
	"net/url"
)

var (
	proxy_addr = "http://127.0.0.1:8081"
	port       = "8083"
)

func handler(w http.ResponseWriter, req *http.Request) {
	// step 1：解析代理地址，并更改请求体的协议和主机
	proxy, _ := url.Parse(proxy_addr)
	req.URL.Scheme = proxy.Scheme // 更改请求协议
	req.URL.Host = proxy.Host     // 更改请求地址

	// step 2：请求下游
	transport := http.DefaultTransport
	resp, err := transport.RoundTrip(req)
	if err != nil {
		log.Println(err)
		return
	}

	// step 3：把响应返回给上游
	for k, vv := range resp.Header {
		for _, v := range vv {
			w.Header().Add(k, v)
		}
	}
	defer resp.Body.Close() // 让Body不再占用内存资源
	bufio.NewReader(resp.Body).WriteTo(w)
}

func main() {
	http.HandleFunc("/", handler)
	log.Println("server starting on port", port)
	if err := http.ListenAndServe(":"+port, nil); err != nil {
		log.Fatal(err)
	}
}
```

* 测试 命令行输入`curl http://127.0.0.1:8083`
> 请求反向代理（127.0.0.1:8083），通过反向代理将请求发送给真正的服务器（127.0.0.1:8081）

### ReverseProxy
* 支持更改内容
* 错误信息回调
* 支持自定义负载均衡
* url重写功能
* 连接池功能
* 支持webSocker服务
* 支持https代理

1. `用ReverseProxy简单实现一个http代理`
```Go
package main

import (
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
)

var addr = "127.0.0.1:8083" // 代理地址

// 用ReverseProxy来简单实现一个代理

func main() {
	rs1 := "http://127.0.0.1:8081/base/" // 真实服务器地址
	url1, err1 := url.Parse(rs1)
	if err1 != nil {
		log.Println(err1)
	}
	proxy := httputil.NewSingleHostReverseProxy(url1)
	log.Println("proxyServer Listening on ", addr)
	// 开启代理监听
	if err := http.ListenAndServe(addr, proxy); err != nil {
		log.Fatal(err)
	}
}
```

2. `修改ReverseProxy来实现可以修改返回内容的代理`
```Go
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
	"strings"
)

var addr = "127.0.0.1:8083" // 代理地址

func NewSingleHostReverseProxy(target *url.URL) *httputil.ReverseProxy {
	targetQuery := target.RawQuery
	director := func(req *http.Request) {
		req.URL.Scheme = target.Scheme
		req.URL.Host = target.Host
		// 将请求路径附加到目标路径下
		req.URL.Path = singleJoiningSlash(target.Path, req.URL.Path)
		if targetQuery == "" || req.URL.RawQuery == "" {
			req.URL.RawQuery = targetQuery + req.URL.RawQuery
		} else {
			req.URL.RawQuery = targetQuery + "&" + req.URL.RawQuery
		}
		if _, ok := req.Header["User-Agent"]; !ok {
			// explicitly disable User-Agent so it's not set to default value
			req.Header.Set("User-Agent", "")
		}
	}
	modifyFunc := func(res *http.Response) error {
		if res.StatusCode == http.StatusOK {
			// 读取body
			oldPayload, err := ioutil.ReadAll(res.Body)
			if err != nil {
				return err
			}
			newPayload := []byte("你好" + string(oldPayload))
			// 产生新body，修改body，并改变contentLength
			res.Body = ioutil.NopCloser(bytes.NewBuffer(newPayload))
			res.ContentLength = int64(len(newPayload))
			res.Header.Set("Content-Length", fmt.Sprintf("%d", res.ContentLength))
		}
		return nil
	}
	return &httputil.ReverseProxy{
		Director:       director,
		ModifyResponse: modifyFunc,
	}
}

func singleJoiningSlash(a, b string) string {
	aslash := strings.HasSuffix(a, "/")
	bslash := strings.HasPrefix(b, "/")
	switch {
	case aslash && bslash:
		return a + b[1:]
	case !aslash && !bslash:
		return a + "/" + b
	}
	return a + b
}

// 用ReverseProxy来简单实现一个代理
func main() {
	rs1 := "http://127.0.0.1:8081/base/" // 真实服务器地址
	url1, err1 := url.Parse(rs1)
	if err1 != nil {
		log.Println(err1)
	}
	proxy := NewSingleHostReverseProxy(url1)
	log.Println("proxyServer Listening on ", addr)
	// 开启代理监听
	if err := http.ListenAndServe(addr, proxy); err != nil {
		log.Fatal(err)
	}
}
```

3. `实现X-Forwarded-For，X-Real-IP`
> X-Forwarded-For：记录最后直连实际服务器之前，整个代理过程；可能会被伪造
> X-Real-IP：请求实际服务器的IP（Client_IP）；不会被伪造 
