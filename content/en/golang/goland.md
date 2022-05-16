---
date: 2021-04-04
description: "Goland实用技巧"
image: "/images/Go.jpg"
title: "Goland"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 快速生成一个模板代码
> 我们平常在goland的go文件中输入一个`main`，再回车就自动生成了`main`函数，类似的还有`init`，`forr`，这些都是goland`实时模板（Live Templates）`的功能

{{< boxmd >}}
* 一次打开 goland > 设置 > 编辑器 > 实时模板
* 可以自己创建一个模板组，也可以直接在gol模板组里面增加一个动态模板
* 比如缩写为`max`，适用于Go 文件，模板文本如下
```Go
func min(a, b $Type$) $Type$ {
    if a < b {
        return a
    }
    return b
}
```
* 我们可以将自己经常写的代码加入到模板组中，比如修改`err`
{{< /boxmd >}}

## 快速生成对应接口方法代码
{{< boxmd >}}
* 按下 生成方法的快捷键（可去设置进行修改），输入需要实现的对应接口（比如:`heap.Interface`）
{{< /boxmd >}}



## 友好的快捷键
{{< boxmd >}}
* `ctrl +`, `ctrl -`折叠，展开代码或者注释
* `doubule shift`（按两次shift），使用最强搜索功能
{{< /boxmd >}}

## 插件
{{< boxmd >}}
* `tabnine`——友好的AI代码补全
* `One Dark theme`——热爱的暗黑模式
{{< /boxmd >}}