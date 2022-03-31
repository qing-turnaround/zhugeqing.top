---
date: 2022-03-30
description: "Linux Shell脚本"
image: "images/linux.jpg"
title: "Linux Shell脚本"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- Linux
series:
- 
---

## 基本操作

* 一般在shell脚本的第一行写上`#!/bin/bash`，如果运行命令是`bash a.sh`，则这一行会被当成注释，而如果就是`./a.sh`来执行，则会解释为使用`bash`来执行。（以`#!`开头）

* shell脚本使用`#`开头来进行注释

* `shell脚本的执行方式`
{{< boxmd >}}
* `bash 1.sh`
* `./1.sh`
* `source 1.sh`
* `. 1.sh`
* 区别1：使用`./1.sh`这种方式执行，需要对文件赋予执行权限，其他不用。
* 区别2：使用`bash 1.sh`，`./1.sh`方式执行，会产生一个子进程来执行，比如sh脚本中是`cd /temp`，而在当前目录执行，执行完成之后，也不会进入`/temp`。而其他两种可以
{{< /boxmd >}}

* `|`
{{< boxmd >}}
* 管道是进程通信的方式之一，能将前一个命令的执行结果传递给后面的命令
* 例：`ls | grep 1*`
{{< /boxmd >}}

* `<`
{{< boxmd >}}
* 输入重定向
* 例： `read zhugeqing < zhugeqing.txt`（将zhugeqing.txt的内容作为$zhugeqing的内容）
{{< /boxmd >}}

* `>`, `>>` , `&>`
{{< boxmd >}}
* 输出重定向
* 例：`echo "zhugeqing" > zhugeqing.txt`（将内容zhugeqing写入并覆盖zhugeqing.txt里面的内容）
* 例：`echo "zhugeqing" >> zhugeqing.txt`（将内容zhugeqing追加到zhugeqing.txt里面）
* 例：`ld &> error.log` （将执行产生的错误写入到文件中;`错误重定向`）
{{< /boxmd >}}

* `输入和输出重定向组合使用`
{{< boxmd >}}
cat > /root/a.txt <<EOF
zhugeqing
EOF
{{< /boxmd >}}

*` 变量赋值`
{{< boxmd >}}
* 变量名只能包含字母，数字，下划线；变量名不能以数字开头
* `变量名=变量值`；例：`a=12345`
* `使用let为变量赋值`；例：`let a=12345`
* `将命令赋给变量`；例：`l=ls`（可以临时将一些比较长的命令赋给一个变量再用于使用）
* `将命令结果赋给变量`；例：`ls_res=$(ls)`（或者使用`` ` `` `` ` ``来代替`$()`） 
* `变量值`有空格等特殊符号需要包含在`" "`活`' '`中；例：`string="123 33"`
{{< /boxmd >}}