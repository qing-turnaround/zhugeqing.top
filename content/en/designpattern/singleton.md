---
date: 2022-10-10
description: "设计模式之单例模式"
image: "images/solo.jpg"
title: "单例模式"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- 设计模式
series:
- 
---

## 使用场景
* 就比如Windows的任务管理器，无法我们按下多少次快捷键，也只会有一个任务管理器窗口
* 也就是一个类，有且只能 创建出一个对象

## 分类
* 单例模式根据实例对象创建的时间 可以分为 `饿汉模式` 和 `懒汉模式`
* `饿汉模式`重点在于`饿`，也就是它在程序运行的时候就会创建好对象，但可能很长时间也不会去使用这个对象，陷入一种饥饿的状态
* `懒汉模式`重点在于`懒`，也就是只有当有人调用创建实例的方法的时候，才会创建对象，给人一种很懒的感觉，我不会主动去创建，得等人叫我去干才干。

## C++实现

### 懒汉模式
```cpp:singleton.cpp
#include "singleton.h"
mutex Singleton::_mtx;
Singleton* Singleton::This = nullptr; // 懒汉式
const Singleton* Singleton::getInstance() {
    // 双重检查
    if(This == nullptr) {
        _mtx.lock(); // 此时还需要再次检测This是否为nullptr，否则会重复new对象
        if(This == nullptr) {
            This = new Singleton;
        }
        _mtx.unlock();
    }
    cout << a << endl;
    This->Print();
    return This;
}
```

```h:singleton.h
#pragma once
#include <iostream>
#include <mutex>
using namespace std;
class Singleton {
public:
    static const Singleton* getInstance();
    static void Print() {
        cout << "This is Print()" << endl;
    }
    static const int a = 100;
private:
    Singleton();
    virtual ~Singleton();
    static Singleton* This;
    // 互斥锁（防止并发访问懒汉模式返回多个不同对象）
    static mutex _mtx;
};
```
```cpp:main.cpp
// 测试单例函数
void SingletonTest() {
    const Singleton* s1 = Singleton::getInstance();
    const Singleton* s2 = Singleton::getInstance();
    s1->Print();
    s2->Print();

    cout << (s1 == s2) << endl;
}
```
### 饿汉模式
```cpp:singleton.cpp
Singleton* Singleton::This = new Singleton; // 饿汉式
const Singleton* Singleton::getInstance() {
    This->Print();
    return This;
}
```


## Go实现

### 懒汉模式
```go:main.go
package main

import (
	"fmt"
	"sync"
)

//Singleton 是单例模式类
type Singleton struct{
	name string
}

var once sync.Once
var singleton *Singleton
// 懒汉模式
func (s Singleton)GetInstance() *Singleton {
	once.Do(func() {
		singleton = &Singleton{name: "singleton"}
	})

	return singleton
}

func (s Singleton)Print() {
	fmt.Println("I am ", s.name)
}


func main() {
	s1 := Singleton{}.GetInstance()
	s1.GetInstance()

	s2 := Singleton{}.GetInstance()
	s2.GetInstance()

	fmt.Println(s1 == s2)
}
```

### 饿汉模式
```go:main.go
package main

import (
	"fmt"
)

//Singleton 是单例模式类
type Singleton struct{
	name string
}

var singleton *Singleton = new(Singleton)
// 饿汉模式
func (s Singleton)GetInstance() *Singleton {
	return singleton
}

func (s Singleton)Print() {
	fmt.Println("I am ", s.name)
}


func main() {
	s1 := Singleton{}.GetInstance()
	s1.GetInstance()

	s2 := Singleton{}.GetInstance()
	s2.GetInstance()

	fmt.Println(s1 == s2)
}
```