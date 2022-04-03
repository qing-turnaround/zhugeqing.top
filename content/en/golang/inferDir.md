---
date: 2022-01-03
description: "推断一个项目的工作目录"
image: "/images/Go.jpg"
title: "推断一个项目的工作目录"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---


## 实现
{{< boxmd >}}
`当项目读取文件时，帮助准确找到文件所在路径，迅速推断出项目根目录`
{{< /boxmd >}}

```Go:infer.go
package infer

import (
	"os"
	"path/filepath"
)

var (
	// 项目主目录
	RootDir string
)

// 推断 Root目录
func InferRootDir() {
	pwd, err := os.Getwd()
	if err != nil {
		panic(err)
	}

	var infer func(string) string
	infer = func(dir string) string {
		if exists(dir + "/template") {
			return dir
		}

		// 查看dir的父目录
		parent := filepath.Dir(dir)
		return infer(parent)
	}

	RootDir = infer(pwd)
}

func exists(dir string) bool {
	// 查找主机是不是存在 dir
	_, err := os.Stat(dir)
	return err == nil || os.IsExist(err)
}
```