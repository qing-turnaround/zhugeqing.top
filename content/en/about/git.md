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
> 将暂存区的内容加入版本历史中（`git commit -am "init"` 可以将已经进行过`追踪（track）`的文件直接提交到版本历史）
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

* `git branch`
> 查看分支 
> `git branch a` 从当前commit创建一个名叫`a`的分支
> `git branch -d a` 删除分支a
> `git branch -v` 查看分之以及对应的commit

* `git checkout`
> 切换分支
> `git checkout master` 切换到`master`分支
> `git checkout -b a` 创建分支a并切换到分支a

* `git commit --amend`
> 修改最新一次commit 的message（编辑文件类似vim） 

* `git rebase -i 83b81348aa04`
> 83b81348aa04为想要修改commit message的上一个分支
> 修改文件的`pick`为`r`，保存退出，再进行修改message，保存退出



* `gitk` 
> 图形化界面工具（查看有中文乱码，需在gitconfig中加入）
{{< boxmd >}}
[gui]
    encoding = utf-8
{{< /boxmd >}}



## git 探秘
* `.git`文件夹用于帮助`git`工具来管理仓库，进入`.git/objects`可以查看对应的`git类型`
* `git分为commit, tree, blob三种类型`，每一个`commit`对应一个或者多个`tree`，一个tree可以对应其他`tree（可以看作是文件夹）`和`blob（相当于是一个文件）`
* `.git/config`文件可以用于配置当前仓库的信息，比如远程仓库，user，email
* `.git/logs`下可以查看各分支commit的哈希值，
* `.git/HEAD`记录当前指向的分支


## git 特殊知识
* 如果使用`git checkout` 切换到某一个commit上而不生成分支，这种`分离HEAD指针`的情况，如果只会再切换到其他分支上，那么将可能丢失在这个commit上后产生的修改（`需要自行记住提交的hash值，再切换回来`），（`HEAD要尽量与分支绑定在一起进行操作`）
* 使用`git cat-file -t`可以查看`git 产生的hash值的类型`，使用`git cat-file -p`可以查看`类型具体的内容`，

{{< figure src="/images/about/git.jpg" >}}




