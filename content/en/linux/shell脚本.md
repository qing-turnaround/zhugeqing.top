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

### `shell脚本的执行方式`
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

### `输入和输出重定向组合使用`
{{< boxmd >}}
cat > /root/a.txt <<EOF
zhugeqing
EOF
`>`  用于覆盖，`>>`用于追加
{{< /boxmd >}}

### `变量赋值`
{{< boxmd >}}
* 变量名只能包含字母，数字，下划线；变量名不能以数字开头
* `变量名=变量值`；例：`a=12345`
* `使用let为变量赋值`；例：`let a=12345`
* `将命令赋给变量`；例：`l=ls`（可以临时将一些比较长的命令赋给一个变量再用于使用）
* `将命令结果赋给变量`；例：`ls_res=$(ls)`（或者使用`` ` `` `` ` ``来代替`$()`） 
* `变量值`有空格等特殊符号需要包含在`" "`活`' '`中；例：`string="123 33"`
{{< /boxmd >}}

### `变量的引用`
{{< boxmd >}}
* $zhugeqing `（zhugeqing为 shell变量）`
* ${zhugeqing} 
* 在shell脚本中，使用shell变量，由于`bash 1.sh`和`./1.sh`这两种方式会生成子进程，会导致变量无法到子进程使用，所有需要通过`export zhugeqing`来导出变量，使得子进程也可以访问
* 使用`unset zhugeqing`来删除变量
{{< /boxmd >}}

### `环境变量`
{{< boxmd >}}
* `$PATH`也就是执行命令的搜索路径，当有命令想在任意路径都能对其进行访问，可以在`/etc/profile`文件中加入`PATH=$PATH:/root`（比如这个命令就在/root/目录下），需要再`source`一下profile文件
* 通过修改变量`$PS1`可以改变shell提示终端符，通过修改`/root/.bashrc`来到达永久生效的效果，比如设置`PS1="[\e[32;40m\u @\h \t \w$]"`（具体搜索——修改linux shell 变量PS1），也可以将变量加入到`/etc/profile`，然后再`source`
{{< /boxmd >}}

### `预定义变量`
{{< boxmd >}}
* `$?`:展示上一条命令的执行结果是否正常，0为正常，1为错误
* `$$`:展示当前进程的进程ID
* `$0`:展示当前的进程名字
{{< /boxmd >}}

### `位置变量`
{{< boxmd >}}
* `$0` `$1`... `${11}`可以用于读取执行shell脚本所带的值
```Shell:1.sh
#!/bin/bash

echo $1
echo $2
```
`source 1.sh 诸葛青 shell`

```Shell:2.sh
#!/bin/bash

echo $1
echo ${2-我就是shell}
```
`source 1.sh 诸葛青`（可以解决传入参数不足，默认使用`我就是shell`）
{{< /boxmd >}}

## 进阶姿势

### `read`
{{< boxmd >}}
* `-p`：输出提示信息
* `-t`：等待用户输入时间， 如果到了时间就不让用户继续输入了（单位为秒）
{{< /boxmd >}}
```Shell:read.sh
#!/bin/bash

read -t 5 -p "请输入您的名字：" name # 等待5s用户输入
name=${name:-诸葛青} # 如果没有输入任何值则赋予变量默认值
echo "您的名字是$name"
```

### `基本语法`
{{< boxmd >}}
1. “$((运算式))”或“$[运算式]”或者 expr m + n //expression 表达式 
2. 注意 expr 运算符间要有空格, 如果希望将 expr 的结果赋给某个变量，使用 temp = \`expr 1 + 2\` 
3. expr m - n 
4. expr \*, /, % 乘，除，取余
{{< /boxmd >}}

### `判断`
{{< boxmd >}}
`判断语法`
1. 单分支
if [ 判断条件 ]
then
代码
fi 

2. 多分支
if [ 判断条件 ]
then
代码
elif [ 判断条件 ]
then
代码
fi 

3. case
case $1 in
"1")
echo "周一" 
;;
"2")
echo "周二"
;;
"3")
echo "周三" 
;;
*)
echo "other"
esac
{{< /boxmd >}}

{{< boxmd >}}
`判断条件`：
1. 字符串比较：`=`
2. 数字比较：
小于：`-lt`（Less than）
小于等于：`-le`（Less than equal）
大于：`-gt`（Greater than）
大于等于：`-ge`（Greater than equal）
不等于：`-ne`（not equal）
3. 按照文件权限进行判断
有读的权限：`-r`
有写的权限：`-w`
有执行的权限：`-x`
4. 按照文件类型进行判断
文件存在且是一个常规的文件：`-f`
文件存在：`-e`
是一个目录：`-d`
{{< /boxmd >}}

### `循环`
1. for循环
```Shell:a.sh
#!/bin/bash 
#从命令行输入一个数 n，统计从 1+..+ n 的值是多少？ 
SUM=0
for i in `seq 1 $1`; do
        SUM=`expr $SUM + $i`
done 
echo $SUM
```

```Shell:a.sh
#!/bin/bash
#从命令行输入一个数 n，统计从 1+..+ n 的值是多少？ 
SUM=0
for ((i = 1; i <= $1; i++))
do
        SUM=`expr $SUM + $i`
done 
echo $SUM
```

2. while循环
```Shell:a.sh
#!/bin/bash
#从命令行输入一个数 n，统计从 1+..+ n 的值是多少？ 
SUM=0
i=0
while [ $i -le $1 ] 
do
        SUM=`expr $SUM + $i`
        i=`expr $i + 1`

done
echo $SUM 
```

### `函数`

```Shell
#!/bin/bash
function HelloWorld() {
    local a="Hello World"
    echo $a
    echo $1
}

HelloWorld $1

a=$'111'
echo $a
```

