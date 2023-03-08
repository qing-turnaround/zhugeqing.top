---
date: 2022-04-04
description: "C++ STL"
image: "images/solo.jpg"
title: "C++ STL"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- c++
series:
- 
---

## 容器

### 分类
* 容器可以按照存放的数据方式，可以分为`序列式容器` 和 `关联式容器`
* `序列式容器`包括 vector, list, deque, 还有是适配器的stack, queue, priority_queue
* `关联式容器`包括 set, multiset, map, multimap

### map 
```h:map_test.h
#pragma once
#include <map>
#include <string>
#include <iostream>
#include <algorithm>
using namespace std;

struct MapDisplay {
    void operator()(pair<string, double> Pair) { // 重载括号运算符
        cout << Pair.first << " " << Pair.second << endl;
    }
};

void MapTest() {
    // 初始化一个Map
    map<string, double> i_d_map;
    // 添加数据
    i_d_map["zhugeqing1"] = 100; // 1
    i_d_map["zhugeqing4"] = 400;
    i_d_map.insert(pair<string, double>("zhugeqing2", 200)); // 2
    i_d_map.insert(map<string, double>::value_type ("zhugeqing3", 300)); // 3


    // 遍历数据
    for_each(i_d_map.begin(), i_d_map.end(), MapDisplay());

    // 查找数据
    auto iter = i_d_map.find("zhugeqing2");
    cout << "find zhugeqin2 Pair is "<< iter->first << " " << iter->second << endl;

    // 移除数据
    i_d_map.erase(iter);
    for_each(i_d_map.begin(), i_d_map.end(), MapDisplay());


    cout << "使用迭代器遍历map" << endl;
    // 使用迭代器遍历，并移除小于300的 pair
    for(iter = i_d_map.begin(); iter != i_d_map.end();) {
        if(iter->second < 300) {
            i_d_map.erase(iter++); // 删除这个pair之后，迭代器失效
        } else {
            iter++;
        }
    }
    
    for(iter = i_d_map.begin(); iter != i_d_map.end(); iter++) {
        cout << "Pair is "<< iter->first << " " << iter->second << endl;
    }

}
```



