---
date: 2020-12-13
description: "关于git 的使用"
image: "images/recommend_site/xingyouji.jpg"
title: "git笔记"
---

## git 配置
* `git config --global user.name "xing-you-ji"`
* `git config --global user.email "2110833194@qq.com"`
> 配置用户名和邮箱，不加--global参数就相当于只对当前目录仓库有效\
> --system表示对系统登录的所有用户有效

* `git config --global --list `
> 查看git当前用户仓库配置，不加--global参数就相当于显示当前目录仓库
> 同样还可以用--system

## git 仓库初始化
* 已经有项目代码，在项目目录执行`git init`
* 还未开始创建仓库，执行`git init project_name`
* `git config --global user.name "xing-you-ji"`
* `git config --global user.email "2110833194@qq.com"`
* 新加入文件，使用`git add .`
> 将最新修改从工作区加入暂存区
* `git commit -m "init"`
> 将暂存区的内容加入版本历史中（`git commit -am "init"`加入暂存区并进行提交）
* `git status`
> 查看`工作区`与`暂存区`状态
* `git add .`
> 将当前目录及子孙目录里的变动都加到暂存区
* `git add -A`
> 将当前仓库所有变动加入暂存区
* `git mv a.txt b.txt`
> 将仓库的a.txt文件修改成b.txt（并将修改加入暂存区）

* `git rm a.txt`
> 将仓库的a.txt文件进行删除（并将修改加入暂存区）

* `git log`
> 查看版本提交历史
> `git log master` 查看指定分支提交历史
> 参数`--oneline`简洁地查看提交历史
> 参数`--all`查看所有分支的提交历史
> 参数`--graph`带图形的查看
> 参数`-n2`查看最近2次提交历史（数字可修改）

* `gitk` 
> 图形化界面工具（查看有中文乱码，需在gitconfig中加入）
{{< boxmd >}}
[gui]
    encoding = utf-8
{{< /boxmd >}}



{{< figure src="/images/about/git.jpg" >}}




