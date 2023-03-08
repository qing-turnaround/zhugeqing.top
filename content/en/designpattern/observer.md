---
date: 2022-10-10
description: "设计模式之观察者模式"
image: "images/solo.jpg"
title: "观察者模式"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- 设计模式
series:
- 
---

## 使用场景
* `观察者模式`也可以被叫做`发布——订阅模式`，只要是场景的发布订阅场景就可以使用
* 举例：一个公司里有几个程序员喜欢在老板离开公司的摸鱼，但是如果被老板回来发现的话，那就完蛋了，所以这几个程序员决定跟前台的姐姐 打好关系，在老板回到公司之后，立马发送消息通知他们，这样这几个程序员就能在摸鱼 与 工作之间反复横跳。而这里的程序员就可以当做`订阅者`，前台当做`发布者`，当前台发布老板回来的消息的时候，程序员们就必须立即作出反应
* `简单的说，当一个对象的改变需要同时改变其他对象的时候，就可以使用观察者模式`
> 来自《大话设计模式》中的观察者模式 故事，订阅者也是观察者；发布者也是通知者，或是主题


## 优缺点
* 优点：
    1. 观察者模式满足“开-闭原则”。主题接口仅仅依赖于观察者接口，而观察者具体是什么，可以自行实现
    2. 耦合的双方都依赖与抽象，而不是依赖具体的实现
* 缺点：
    1. 如果通知的观察者较多，将会耗费不少时间
    2. 如果观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩
## C++实现

### 抽象发布者接口
```h:Subject.h
#pragma once
#include <string>
#include "Observer.h"

// 抽象发布者——通知者（抽象主题）
class Subject {
public:
    virtual void Attach(Observer * observer) = 0;  // 注册订阅者
    virtual void Detach(Observer * observer) = 0;  // 注销订阅者
    virtual void Notify(string) = 0;  // 通知订阅者
private:
};
```


### 具体发布者——前台
```h:ConcreteSubject.h
#pragma once
#include <list>
#include "Observer.h"
#include "IObject.h"
using namespace std;

// 具体发布者——比如前台
class ConcreteSubject: public IObject{
public:
    // 注册订阅者
    void Attach(Observer* observer) {
        observers.push_back(observer);
    }

    // 注销订阅者
    void Detach(Observer* observer) {
        observers.remove(observer);
    }

    // 通知订阅者(一次性通知所有)
    void Notify(string event) {
        for(auto it : observers) {
            it->Update(event);
        }
    }

private:
    list<Observer *> observers;
};
```

### 抽象订阅者(抽象观察者)
```h:Observer.h
#pragma once
#include <string>
using namespace std;
// 抽象类观察者
class Observer {
public:
    virtual void Update(string) = 0; // 收到通知，做出行动

};
```


### 具体订阅者——程序员
```h:Programmer.h
#pragma once
#include <string>
#include <iostream>
#include "Observer.h"
#include "ConcreteSubject.h"
using namespace std;

// 具体观察者——可以根据不同的岗位处理，比如这里是程序员，还可以是产品经理
class Programmer: public Observer {
public:
    Programmer(string &name, Subject * mySubject) :
        my_name(name),
        my_subject(mySubject)
        {}

    void Update(string event) override {
        my_event = event;
        cout << my_name << "收到前台通知：" << my_event << endl;
    }

private:
    string my_name; // 观察者名字
    string my_event; // 观察者当前收到通知
    Subject* my_subject; // 订阅的主题，或者是 关联的 发布者
};

```

### 具体订阅者——产品经理
```h:ProductManager.h

#pragma once
#include <string>
#include <iostream>
#include "Observer.h"
#include "Subject.h"

using namespace std;

// 具体观察者——可以根据不同的岗位处理，比如这里是程序员，还可以是产品经理
class ProductManager: public Observer { // 产品经理
public:
    ProductManager(string &name, Subject* mySubject) :
        my_name(name),
        my_subject(mySubject)
        {}

    void Update(string event) override {
        my_event = event;
        cout << my_name << "收到前台通知：" << my_event << endl;
    }

private:
    string my_name; // 观察者名字
    string my_event; // 观察者当前收到通知
    Subject* my_subject; // 订阅的主题，或者是 关联的 发布者
};

```

### 客户端执行
```cpp:main.cpp
#include "ConcreteSubject.h"
#include "Programmer.h"
#include "ProductManager.h"
int main()
{
    // 创建一个主题（通知者），前台
    auto subject = new ConcreteSubject();
    string name1 = "麦当";
    string name2 = "咕咚";
    string name3 = "笛亚";
    // 程序员
    auto* ob1 = new Programmer(name1, subject);
    // 程序员——观察者
    auto* ob2 = new Programmer(name2, subject);

    // 产品经理
    auto* ob3 = new ProductManager(name3, subject);

    // 观察者1 订阅 主题
    subject->Attach(ob1);
    // 观察者2 订阅 主题
    subject->Attach(ob2);
    // 观察者3 订阅 主题
    subject->Attach(ob3);

    // 通知程序员
    subject->Notify("老板走了，开始追求自由吧！");

    // 通知者程序员
    subject->Notify("老板回来了，赶紧开始工作");

    // 注销：下班了，不需要前台来通知了
    subject->Detach(ob1);
    subject->Detach(ob2);
    subject->Detach(ob3);

    return 0;
}
```