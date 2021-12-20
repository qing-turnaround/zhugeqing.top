---
date: 2021-12-01
description: "Go语言html/template的学习"
image: "/images/Go.jpg"
title: "Go语言template库"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 模板引擎的使用步骤
1. 定义模板文件
2. 解析模板文件
```Go
//常用方法
func (t *Template) Parse(src string) (*Template, error)
func ParseFiles(filenames ...string) (*Template, error)
func ParseGlob(pattern string) (*Template, error)
```
3. 模板渲染
```Go
//常用方法
func (t *Template) Execute(wr io.Writer, data interface{}) error
func (t *Template) ExecuteTemplate(wr io.Writer, name string, data interface{}) error
```


## 示例

模板文件`go.tmpl`
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>template示例</title>
</head>
<body>
<p>你好呀！{{.}}</p>
</body>
</html>
```

代码文件`main.go`
```Go
package main

import (
	"fmt"
	"html/template"
	"net/http"
)

func example(w http.ResponseWriter, r *http.Request) {
	// 解析指定文件生成模板对象
	tmpl, err := template.ParseFiles("./go.tmpl")
	if err != nil {
		fmt.Println("create template failed, err:", err)
		return
	}
	// 通过执行Execute将data渲染到模板，并写入到ResponseWriter
	tmpl.Execute(w, "诸葛青")
}

func main() {
	http.HandleFunc("/", example)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("HTTP server failed,err:", err)
		return
	}
}
```

## 模板语法

### {{.}}
模板语法都包含在`{{和}}`中间，其中`{{.}}`中的点表示当前对象。
当传入一个结构体对象时，可以根据`.`来访问结构体的对应字段：

代码文件`main.go`
```Go
package main

import (
	"fmt"
	"html/template"
	"net/http"
)

func example(w http.ResponseWriter, r *http.Request) {
	// 解析指定文件生成模板对象
	tmpl, err := template.ParseFiles("./go.tmpl")
	if err != nil {
		fmt.Println("create template failed, err:", err)
		return
	}
	// 通过执行Execute将data渲染到模板，并写入到ResponseWriter
	tmpl.Execute(w, map[string]interface{}{
		"Name":     "诸葛青",
		"Age":      19,
		"favorite": "nothing",
	})
}
func main() {
	http.HandleFunc("/", example)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("HTTP server failed,err:", err)
		return
	}
}

```

模板文件`go.tmpl`
```HTML
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>template示例</title>
</head>
<body>
<p>姓名：{{.Name}}</p>
<p>年龄：{{.Age}}</p>
<p>最喜爱的：{{.favorite}}</p>
</body>
</html>
```

### 注释
`{{/* a comment */}}`
注释，执行时会忽略。可以多行。注释不能嵌套，并且必须紧贴分界符始止。

### pipeline
`pipeline`是指产生数据的操作。比如`{{.}}`、`{{.Name}}`等。Go的模板语法中支持使用管道符号`|`链接多个命令，用法和unix下的管道类似：`|`前面的命令会将运算结果(或返回值)传递给后一个命令的最后一个位置。

* 注意 : 并不是只有使用了`|`才是`pipeline`。Go的模板语法中，`pipeline`的概念是传递数据，只要能产生数据的，都是`pipeline`。

### 变量
在模版里可以初始化一个变量来捕获传入模板的数据或其他语句生成的结果。初始化语法如下：
`$variable := pipeline`

### 移除空格
使用`{{-`语法去除模板内容左侧的所有空白符号， 使用`-}}`去除模板内容右侧的所有空白符号。
{{< alert theme="info" >}}
{{- .Name -}}  
{{< /alert >}}
* 注意：`-`要紧挨`{{`和`}}`，同时与模板值之间需要使用空格分隔。

### 条件判断
Go模板语法中的条件判断有以下几种:
{{< alert theme="info" >}}
{{if pipeline}} T1 {{end}}

{{if pipeline}} T1 {{else}} T0 {{end}}

{{if pipeline}} T1 {{else if pipeline}} T0 {{end}}
{{< /alert >}}

### range
Go的模板语法中使用`range`关键字进行遍历，有以下两种写法，其中pipeline的值必须是数组、切片、字典或者通道。
```HTML
{{range $key, $val := .}}
<p>{{$key}} - {{$val}}</p>
{{end}}
```
> 果pipeline的值其长度为0，不会有任何输出

```HTML
{{range $key, $val := .}}
<p>{{$key}} - {{$val}}</p>
{{else }}}
<p>长度为0</p>
{{end}}
```
> 如果pipeline的值其长度为0，则会执行else

### with
> 相当于构建一个作用域
```Linux
{{with pipeline}} T1 {{end}}
如果pipeline为empty不产生输出，否则将dot设为pipeline的值并执行T1。不修改外面的dot。

{{with pipeline}} T1 {{else}} T0 {{end}}
如果pipeline为empty，不改变dot并执行T0，否则dot设为pipeline的值并执行T1。
```

### 预定义函数
执行模板时，函数从两个函数字典中查找：首先是模板函数字典，然后是全局函数字典。一般不在模板内定义函数，而是使用Funcs方法添加函数到模板里。

预定义的全局函数如下：
```linux
and
    函数返回它的第一个empty参数或者最后一个参数；
    就是说"and x y"等价于"if x then y else x"；所有参数都会执行；
or
    返回第一个非empty参数或者最后一个参数；
    亦即"or x y"等价于"if x then x else y"；所有参数都会执行；
not
    返回它的单个参数的布尔值的否定
len
    返回它的参数的整数类型长度
index
    执行结果为第一个参数以剩下的参数为索引/键指向的值；
    如"index x 1 2 3"返回x[1][2][3]的值；每个被索引的主体必须是数组、切片或者字典。
print
    即fmt.Sprint
printf
    即fmt.Sprintf
println
    即fmt.Sprintln
html
    返回其参数文本表示的HTML逸码等价表示。
urlquery
    返回其参数文本表示的可嵌入URL查询的逸码等价表示。
js
    返回其参数文本表示的JavaScript逸码等价表示。
call
    执行结果是调用第一个参数的返回值，该参数必须是函数类型，其余参数作为调用该函数的参数；
    如"call .X.Y 1 2"等价于go语言里的dot.X.Y(1, 2)；
    其中Y是函数类型的字段或者字典的值，或者其他类似情况；
    call的第一个参数的执行结果必须是函数类型的值（和预定义函数如print明显不同）；
    该函数类型值必须有1到2个返回值，如果有2个则后一个必须是error接口类型；
    如果有2个返回值的方法返回的error非nil，模板执行会中断并返回给调用模板执行者该错误； 
```

### 比较函数
布尔函数会将任何类型的零值视为假，其余视为真。
下面是定义为函数的二元比较运算的集合：
```Linux
    eq      如果arg1 == arg2则返回真
    ne      如果arg1 != arg2则返回真
    lt      如果arg1 < arg2则返回真
    le      如果arg1 <= arg2则返回真
    gt      如果arg1 > arg2则返回真
    ge      如果arg1 >= arg2则返回真 
```

为了简化多参数相等检测，eq（只有eq）可以接受2个或更多个参数，它会将第一个参数和其余参数依次比较，返回下式的结果：
` {{eq arg1 arg2 arg3}} `
`比较函数只适用于基本类型（或重定义的基本类型，如”type Celsius float32”）。但是，整数和浮点数不能互相比较。`

### 自定义函数
Go的模板支持自定义函数。         
代码文件`main.go`
```Go
package main

import (
	"fmt"
	"html/template"
	"net/http"
)

func example(w http.ResponseWriter, r *http.Request) {
	//定义一个模版引擎
	t := template.New("go.tmpl")
	//在模版中自定义函数（如下函数名为"夸奖"）
	t.Funcs(template.FuncMap{"夸奖": func(name string) (string, error) {
		return name + "持续且努力", nil
	}})
	// 解析指定文件生成模板对象
	_, err := t.ParseFiles("./go.tmpl")
	if err != nil {
		fmt.Println("create template failed, err:", err)
		return
	}
    //渲染模版
	t.Execute(w, "诸葛青")

}
func main() {
	http.HandleFunc("/", example)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("HTTP server failed,err:", err)
		return
	}
}
```
模版文件`go.tmpl`
```HTML
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>template示例</title>
</head>
<body>
{{夸奖 .}}
</body>
</html>

```


### 嵌套template
可以在template中嵌套其他的template。这个template可以是单独的文件，也可以是通过define定义的template。

模版文件1`go1.tmpl`
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>tmpl test</title>
</head>
<body>

<h1>测试嵌套template语法</h1>
<hr>
{{template "go2.tmpl"}}
<hr>
<p>我是 {{- . -}}}</p>
</body>
</html>

{{ define "go3.tmpl"}}
    <ol>
        <li>吃饭</li>
        <li>睡觉</li>
        <li>打豆豆</li>
    </ol>
{{end}}
```

模版文件2`go2.tmpl`
```HTML
<ul>
    <li>注释</li>
    <li>日志</li>
    <li>测试</li>
</ul>
```

代码文件`main.go`
```Go
package main

import (
	"fmt"
	"html/template"
	"net/http"
)

func tmplDemo(w http.ResponseWriter, r *http.Request) {
    //解析模版
    tmpl, err := template.ParseFiles("./go.tmpl", "./go2.tmpl")
	if err != nil {
		fmt.Println("create template failed, err:", err)
		return
	}
	user := struct {
		name string
		sex  string
		Age  int
	}{"诸葛青", "男", 19}
    //渲染模版
	tmpl.Execute(w, user)
}

func main() {
	http.HandleFunc("/template", tmplDemo)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("HTTP server failed,err:", err)
		return
	}
}
```

### block
``{{block "name" pipeline}} T1 {{end}}``
> `block`是定义模板`{{define "name"}} T1 {{end}}`和执行`{{template "name" pipeline}}`缩写，典型的用法是定义一组根模板，然后通过在其中重新定义块模板进行自定义。

定义一个根模板`base.tmpl`，内容如下：
```HTML
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>Go Templates</title>
</head>
<body>
<div class="container-fluid">
    {{block "content" . }}{{end}}
</div>
</body>
</html>
```

然后定义一个`index.tmpl`，”继承”`base.tmpl`：
```HTML
{{/*加 . 会继承根模版的数据，反之，则不会*/}}
{{template "base.tmpl" .}}

{{define "content"}}
    <div>我是content模版，我被另外一个模版找到了</div>
    <p>你好!{{.}}</p>
{{end}}
```

代码文件
```Go
package main

import (
	"fmt"
	"html/template"
	"net/http"
)

func index(w http.ResponseWriter, r *http.Request) {
	tmpl, err := template.ParseGlob("*.tmpl")
	if err != nil {
		fmt.Println("create template failed, err:", err)
		return
	}
	//选择模版文件进行渲染
	err = tmpl.ExecuteTemplate(w, "base.tmpl", "诸葛青")
	if err != nil {
		fmt.Println("render template failed, err:", err)
		return
	}
}

func main() {
	http.HandleFunc("/index", index)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("HTTP server failed,err:", err)
		return
	}
}
```

### 修改默认的标识符
``template.New("go.tmpl").Delims("{[", "]}").ParseFiles("./go.tmpl")``

代码文件`main.go`
```Go
package main

import (
	"fmt"
	"html/template"
	"net/http"
)

func test1(w http.ResponseWriter, r *http.Request) {
	t, err := template.New("go.tmpl").
		Delims("{[", "]}").
		ParseFiles("./go.tmpl")
	if err != nil {
		fmt.Println(err)
		return
	}

	err = t.Execute(w, "诸葛青")
	if err != nil {
		fmt.Println(err)
	}
}

func main() {

	http.HandleFunc("/test1", test1)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Println(err)
	}
}
```

模版文件`go.tmpl`
```HTML
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>tmpl test</title>
</head>
<body>
<h1>测试修改默认的标示符</h1>
<p>我是{[ . ]}</p>
</body>
</html>
```