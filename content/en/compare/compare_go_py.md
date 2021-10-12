---
date: 2021-10-07
description: "人生如棋，我愿为卒，行动虽慢，可谁曾见我后退半步？"
image: "images/recommend_site/xingyouji.jpg"
title: "Golang 与 Python语法比较"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- 
series:
- Programming language comparison
---

## 1.打印"Hello World"

{{< codes golang python>}}

  {{< code >}}

  ```golang
package main

import "fmt"

func main() {
	fmt.Println("Hello World")
}
  ```
  {{< /code >}}

  {{< code >}}

  ```python
print("Hello World")

# or

print('Hello World')
  ```
  {{< /code >}}
{{< /codes >}}

## 2.打印"Hello"十次



{{< codes golang python>}}
  {{< code >}}

  ```golang
package main

import (
	"fmt"
    "strings"
)

func main() {
	for i := 0; i < 10; i++ {
		fmt.Println("Hello")
	}

    //or
    fmt.Println(strings.Repeat("Hello\n", 10))
}
  ```

  {{< /code >}}
	
  {{< code >}}

  ```python
# '_'为占位符
for _ in range(10):
    print("Hello")
  
# or

print("Hello\n"*10)

  ``` 
  {{< /code >}}
{{< /codes >}}


## 3.创建一个过程

>就像一个不返回任何值的函数，例如打印到标准输出

{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

func finish(name string) {
	fmt.Println("My job here is done. Good bye " + name)
}

func main() {
	finish("诸葛青")
}
/*
package main

import "fmt"

func main() {
    finish := func(name string) {
        fmt.Println("My job here is done. Good bye " + name)
    }
	
    finish("诸葛青")
}

*/
```

{{< /code >}}
  
{{< code >}}

```python
def finish(name):
    print(f'My job here is done. Goodbye {name}')


finish('诸葛青')

``` 
{{< /code >}}
{{< /codes >}}


## 4.创建一个函数

{{< codes golang python>}}
{{< code >}}

```golang
package main

import (
	"fmt"
)

func square(x int) int {
	return x * x
}

func main() {
	n := square(9)
	fmt.Println(n)
}
```

{{< /code >}}
  
{{< code >}}

```python
# def square(x):
#     return x*x


# x**y 表示x的y次幂
def square(x):
    return x**2


print(square(9))
``` 
{{< /code >}}
{{< /codes >}}


## 5.创建二维点数据结构

{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

type Point struct {
	x, y float64
}

func main() {
	p1 := Point{}
	p2 := Point{2.1, 2.2}
	p3 := Point{
		y: 3.1,
		x: 3.2,
	}
	p4 := &Point{
		x: 4.1,
		y: 4.2,
	}

	fmt.Println(p1)
	fmt.Println(p2)
	fmt.Println(p3)
	fmt.Println(p4)
}

```

{{< /code >}}
  
{{< code >}}

```python
class Point:
    x: float
    y: float


point1 = Point()
point1.x, point1.y = 1.01, 1.001
print(point1.x, point1.y)

``` 
{{< /code >}}
{{< /codes >}}


## 6.迭代列表值

{{< codes golang python>}}
{{< code >}}

```golang
package main

import (
	"fmt"
)

func main() {	
    items := []int{11, 22, 33}	
    for _, x := range items {	
        doSomething(x)	
    }
}

func doSomething(i int) {
	fmt.Println(i)
}
```

{{< /code >}}
  
{{< code >}}

```python
def doSomething(i):
    print(i)


items = [11, 22, 33]
for i in items:
    doSomething(i)
``` 
{{< /code >}}
{{< /codes >}}

## 7.迭代列表值和下标
{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

func main() {
	items := []string{
		"oranges",
		"apples",
		"bananas",
	}

	for i, x := range items {
		fmt.Printf("Item %d = %v \n", i, x)
	}
}
```

{{< /code >}}
  
{{< code >}}

```python
items = ["oranges", "apples", "bananas", ]

# Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身

for i, x in enumerate(items):
    print(i, x)
``` 
{{< /code >}}
{{< /codes >}}


## 8.初始化一个map（映射）


{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

func main() {
	x := map[string]int{"one": 1, "two": 2}

	fmt.Println(x)
}
```

{{< /code >}}
  
{{< code >}}

```python
x = {"one": 1, "two": 2}
print(x)
``` 
{{< /code >}}
{{< /codes >}}

## 9.创建一棵二叉树


{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

type BinTree struct {
	Value int
	Left  *BinTree
	Right *BinTree
}

//中序遍历
func inorder(root *BinTree) {
	if root == nil {
		return
	}

	inorder(root.Left)
	fmt.Printf("%d ", root.Value)
	inorder(root.Right)
}

func main() {
	root := &BinTree{1, nil, nil}
	root.Left = &BinTree{2, nil, nil}
	root.Right = &BinTree{3, nil, nil}
	root.Left.Left = &BinTree{4, nil, nil}
	root.Left.Right = &BinTree{5, nil, nil}
	root.Right.Right = &BinTree{6, nil, nil}
	root.Left.Left.Left = &BinTree{7, nil, nil}

	inorder(root)
}
```

{{< /code >}}
  
{{< code >}}

```python
# 中序遍历
def inorder(self):
    if self == None:
        return
    inorder(self.left)
    print(self.data)
    inorder(self.right)


class Node:
    def __init__(self, data):
        self.data = data
        self.left = None
        self.right = None


root = Node(1)
root.left = Node(2)
root.right = Node(3)
inorder(root)


  ``` 
{{< /code >}}
{{< /codes >}}

## 10.随机化一个列表


{{< codes golang python>}}
{{< code >}}

```golang
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	x := []string{"a", "b", "c", "d", "e", "f", "g", "h"}

	rand.Shuffle(len(x), func(i, j int) {
		x[i], x[j] = x[j], x[i]
	})

	fmt.Println(x)
}
```

{{< /code >}}
  
{{< code >}}

```python
import random
list = ["a", "b", "c", "d", "e", "f", "g", "h"]
random.shuffle(list)
print("Reshuffled list : ",  list)

random.shuffle(list)
print("Reshuffled list : ",  list)

``` 
{{< /code >}}
{{< /codes >}}

## 11.从列表中随机选取一个元素


{{< codes golang python>}}
{{< code >}}

```golang
package main

import (
	"fmt"
	"math/rand"
)

var x = []string{"bleen", "fuligin", "garrow", "grue", "hooloovoo"}

func main() {
	//Intn取[0,len(x))
	fmt.Println(x[rand.Intn(len(x))])
}

```

{{< /code >}}
  
{{< code >}}

```python
import random

list = ["a", "b", "c", "d", "e", "f", "g", "h"]

print(random.choice(list))

``` 
{{< /code >}}
{{< /codes >}}

## 12.查找一个元素是否在列表中

{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

func Contains(list []T, x T) bool {
	for _, item := range list {
		if item == x {
			return true
		}
	}
	return false
}

type T string

func main() {
	list := []T{"a", "b", "c"}
	fmt.Println(Contains(list, "b"))
	fmt.Println(Contains(list, "z"))
}

```

{{< /code >}}
  
{{< code >}}

```python
import random

list = ["a", "b", "c", "d", "e", "f", "g", "h"]

print("404" in list)

or 

print(list.__contains__("a"))

``` 
{{< /code >}}
{{< /codes >}}

## 13.迭代一个映射的键和值


{{< codes golang python>}}
{{< code >}}

```golang
package main

import "fmt"

func main() {
	mymap := map[string]int{
		"one":   1,
		"two":   2,
		"three": 3,
		"four":  4,
	}

	for k, x := range mymap {
		fmt.Println("Key =", k, ", Value =", x)
	}
}

```

{{< /code >}}
  
{{< code >}}

```python
dict = {1: "哈", 2: "哈哈", 3: "哈哈哈"}

for i, v in dict.items():
    print(i, v)

``` 
{{< /code >}}
{{< /codes >}}


## 14.在 [a..b) 中选取一个随机浮点数


{{< codes golang python>}}
{{< code >}}

```golang
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	x := pick(-2.0, 6.5)
	fmt.Println(x)
}

func pick(a, b float64) float64 {
	return a + (rand.Float64() * (b - a))
}
```

{{< /code >}}
  
{{< code >}}

```python
import random
x = random.uniform(1, 5)
print(x)

``` 
{{< /code >}}
{{< /codes >}}

## 15.在 [a..b] 中选取一个随机整数


{{< codes golang python>}}
{{< code >}}

```golang
package main

import (
	"fmt"
	"math/rand"
)

func main(){
	x := pick(3, 4)

	fmt.Println(x)
}

func pick(a, b int) int {
	return a + rand.Intn(b-a+1)
}

```

{{< /code >}}
  
{{< code >}}

```python
import random
x = random.randint(1, 5)
print(x)

``` 
{{< /code >}}
{{< /codes >}}

## 16.深度优先遍历一颗二叉树


{{< codes golang python>}}
{{< code >}}

```golang
package main

import . "fmt"

type key string
type value string

type BinTree struct {
	Key   key
	Deco  value
	Left  *BinTree
	Right *BinTree
}

func (bt *BinTree) Dfs(f func(*BinTree)) {
	if bt == nil {
		return
	}
	bt.Left.Dfs(f)
	f(bt)
	bt.Right.Dfs(f)
}

func main() {
	a := []key{"d", "a", "dd", "e", "h", "gg", "f", "b", "n", "v"}
	tree := &BinTree{Key: a[0]}
	for _, str := range a[1:] {
		tree.Insert(str, value(""))
	}
	tree.Dfs(NodePrint)
}

func (bt *BinTree) Insert(x key, v value) {
	if x < bt.Key {
		if bt.Left == nil {
			bt.Left = &BinTree{Key: x, Deco: v}
		} else {
			bt.Left.Insert(x, v)
		}
	} else {
		if bt.Right == nil {
			bt.Right = &BinTree{Key: x, Deco: v}
		} else {
			bt.Right.Insert(x, v)
		}
	}
}

func NodePrint(node *BinTree) {
	Println(node.Key)
}	
```

{{< /code >}}
  
{{< code >}}

```python
class Binary:
    def __init__(self, data):
        self.data = data
        self.left = None
        self.right = None


def dfs(root):
    if root == None:
        return
    dfs(root.left)
    print(root.data)
    dfs(root.right)


def insert(root, v):
    if root.data > v:
        if root.left == None:
            root.left = Binary(v)
        else:
            insert(root.left, v)
    else:
        if root.right == None:
            root.right = Binary(v)
        else:
            insert(root.right, v)


list = [1, 2, 3, 5, 6, 7]

root = Binary(4)
for i in list:
    insert(root, i)

dfs(root)

``` 
{{< /code >}}
{{< /codes >}}