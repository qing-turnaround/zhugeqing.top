---
date: 2022-04-04
description: "C++ 多线程"
image: "images/solo.jpg"
title: "C++ 多线程"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- c++
series:
- 
---

## 创建一个线程

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Test1(const char* i) {
    cout << "I am thread " << i << endl;
}

int main() {
    // 创建线程t1
    thread t1(Test1, "t1");
    // 创建线程t2
    thread t2(Test1, "t2");

    // 等待线程执行完成
    t1.join();
    // 等待线程执行完成
    t2.join();


    return 0;
}
```

## 使用互斥锁控制并发

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Test1(const char* num, mutex& mu) {
    for(int i = 0; i < 10; i++) {
        mu.lock();
        cout << "I am thread " << num << " and count " << i << endl;
        mu.unlock();
    }
}

int main() {
    mutex mu;
    // 创建线程t1
    auto num1 = "number 1", num2 = "number 2";
    thread t1(Test1, num1, ref(mu));
    // 创建线程t2
    thread t2(Test1, num2, ref(mu));

    // 等待线程执行完成
    t1.join();
    // 等待线程执行完成
    t2.join();


    return 0;
}
```







docker run -e SESSION_TOKEN="$(echo $SESSION_TOKEN)" --rm -it registry.cn-hangzhou.aliyuncs.com/sunshanpeng/wechaty-chatgpt:0.0.8