---
date: 2020-12-13
description: "关于git 的使用"
image: "images/recommend_site/xingyouji.jpg"
title: "git笔记"
---
## 1.创建仓库命令

1.创建版本库(初始化仓库) ------ git init 

2.拷贝一份远程仓库，也就是下载一个项目  ------ git clone

## 2.提交与修改

1.添加文件到暂存区 ------ git add

2.查看仓库当前的状态，显示有变更的文件 ------ git status

3.比较文件的不同，即暂存区和工作区的区别 ------ git diff

4.提交暂存区到本地仓库 ------ git commit

7.回退版本 ------ git reset

8.删除工作区的文件 ------ git rm

9.移动或者重命名工作区的文件 ------ git mv

### 2.1提交日志 

1.查看历史提交记录 ------ git log

2.以列表形式查看指定文件的历史修改记录 ------ git blame <file>

### 2.2远程操作

1.远程仓库操作 ------ git remote

2.从远程获取代码库 ------ git fetch

3.下载远程代码并合并 ------ git pull

4.上传远程代码并合并 ------ git push 

## 3.git 分支管理

1.列出分支 ------ git branch(无参数时)

2.创建新分支 ------ git branch branchname(branchname为想要创建的分支名)

3.切换到分支 ------ git checkout branchname(branchname为想要切换到的分支名)

4.创建新分支并切换到该分支下 ------ git checkout -b branchname(branchname为想要创建的分支名)
 
5.删除分支 ------ git branch -d branchname(branchname为想要删除的分支名)

6.合并分支 ------ git merge /git rebase
##### rebase优缺点
##### 优点：
##### Rebase 使你的提交树变得很干净, 所有的提交都在一条线上
##### 缺点：
##### Rebase 修改了提交树的历史 
##### merge优缺点
##### 优点:
##### merge可以保留提交历史
##### 缺点：
##### merge使你的提交树变得复杂，提交由多条线相连

7.解决冲突 ------ 当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

## 4.git 查看提交历史

1.查看历史提交记录 ------ git log

2.以列表形式查看指定文件的历史修改记录 ------ git blame <file>

## 5.git 标签

1.查看所有标签 ------ git tag

2.为当前分支创建标签 ------ git tag -a v1.0(不带"-a"也行,但建议带上)


3.创建带有说明的标签 ------ git tag -a v0.1 -m "runoob.com标签"（用-a指定标签名，-m指定说明文字：）

4.查看标签说明文字 ------ git show v0.1

##### (注意：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。)

## 6.Git 修改本地和查看本地属性

### 1.git config --local --list(查看本地设置)
### 2.git config --global user.name/email(修改本地属性,加不加--global区别在于是否是修改全局变量)

## 7.git 远程仓库(Github)

### 1添加远程库 ------ git remote add origin [url](origin 为给远程库取的一个别名)

###### 由于你的本地 Git 仓库和 GitHub 仓库之间的传输是通过SSH加密的，所以我们需要配置验证信息：
###### 使用以下命令生成 SSH Key：ssh-keygen -t rsa -C "youremail@example.com" ######
###### 后面的 your_email@youremail.com 改为你在 Github 上注册的邮箱，之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。成功的话会在 ~/ 下生成 .ssh 文件夹，进去，打开 id_rsa.pub，复制里面的 key。回到 github 上，进入 Account => Settings（账户配置）。左边选择 SSH and GPG keys，然后点击 New SSH key 按钮,title 设置标题，可以随便填，粘贴在你电脑上生成的 key ######

### 2查看当前的远程库 ------ git remote -v

### 3拉取远程仓库 

1.从远程仓库下载新分支与数据 ------ git fetch

2从远端仓库提取数据并尝试合并到当前分支 ------ git merge

### 4推送到远程仓库 ------ git push origin branch(branch为远程仓库的分支名)

### 5删除远程仓库 ------ git remote rm origin

## 8.git 服务器搭建

### 1.安装 git ------sudo apt-get install git(ubuntu下的下载,windows可上网查询)
##### 接下来我们 创建一个用户用来运行git服务的用户
###### $ sudo adduser zhugeqing

### 2.创建证书登录
###### 虽然是私有的Git服务器，但是也不能允许主机随意向Git服务器推送代码。因此，必须将需要使用Git服务器，即需要登录到Git服务器的主机的公钥（即id_rsa.pub文件）导入Git服务器的/home/git/.ssh/authorized_keys文件里，一行一个

###3.初始化Git仓库 

##### 如果我们使用/tmp/xingyouji.git做为远程仓库，则需要在/tmp目录下执行：
###### sudo git init --bare xingyouji.git

### 4.克隆仓库 

##### git clone git@192.168.45.4:/home/gitrepo/xingyouji.git

###### 192.168.45.4 为 Git 所在服务器 ip ，你需要将其修改为你自己的 Git 服务 ip，这样我们的 Git 服务器安装就完成。


## 9.[Git Gitee（码云)](https://gitee.com/)

###### 大家都知道国内访问 Github 速度比较慢，很影响我们的使用。如果你希望体验到 Git 飞一般的速度，可以使用国内的 Git 托管服务——Gitee（gitee.com），

###### Gitee 提供免费的 Git 仓库，还集成了代码质量检测、项目演示等功能。对于团队协作开发，Gitee 还提供了项目管理、代码托管、文档管理的服务，5 人以下小团队免费  。 ######

1.我们先在 Gitee 上注册账号并登录后，然后上传自己的 SSH 公钥

2.其他操作根据需求操作，与上述操作无大区别


{{< figure src="/images/about/git.jpg" >}}




