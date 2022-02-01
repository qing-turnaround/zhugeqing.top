---
date: 2021-12-11
description: "mysql-go的学习"
image: "/images/Go.jpg"
title: "Go语言对Mysql进行操作"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## 选择框架
选择使用[go-sql-driver](https://github.com/go-sql-driver/mysql)和[sqlx](github.com/jmoiron/sqlx)（可以理解为database/sql是sqlx的子集）

## 快速使用

```Go
package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

// 定义一个全局对象db
var db *sqlx.DB

// 定义一个初始化数据库的函数
func initDB() (err error) {
	// DSN:Data Source Name
	//DSN格式为：[username[:password]@][protocol[(address)]]/dbname[?param1=value1&...&paramN=valueN]
	dsn := "root:root@tcp(127.0.0.1:3306)/test"
	//"也可以使用mustConnect，如果连接不成功，直接panic"
	db, err = sqlx.Connect("mysql", dsn) //封装了open和ping，不需要再执行ping来进行真正的连接
	if err != nil {
		return err
	}
	fmt.Println("连接mysql成功")
	db.SetMaxOpenConns(200) //最大连接数（默认）
	db.SetConnMaxIdleTime(20) //最大空闲连接数（20个数据库连接随时待命）
	return nil
}

func main() {
	err := initDB() // 调用输出化数据库的函数
	if err != nil {
		panic(err)
	}

	defer db.Close()
}
```


## 准备工作
1. 在Mysql中建立名字为`test`的数据库
> create database test;
> use test;

2. 建立表
```Mysql
CREATE TABLE `person` (
    `user_id` int(11) NOT NULL AUTO_INCREMENT,
    `username` varchar(260) DEFAULT NULL,
    `sex` varchar(260) DEFAULT NULL,
    `email` varchar(260) DEFAULT NULL,
    PRIMARY KEY (`user_id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

CREATE TABLE place (
    country varchar(200),
    city varchar(200),
    telcode int
)ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

3. SetMaxOpenConns

## Insert操作

```Go
//插入数据
func insertDemo() {
	//sql执行语句
	sqlStr := "insert into person(username, sex, email)values(?, ?, ?)"
	//插入数据
	r, err := db.Exec(sqlStr, "诸葛青", "男", "12345@qq.com")
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	id, err := r.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}

	fmt.Println("insert succ:", id)
}
```


## Select操作
```Go
//查询数据
func selectDemo() {
	//存储查询结果的结构体
	var person []Person
	//sql执行语句
	sqlStr := "select user_id, username, sex, email from person where user_id =?"
	user_id := 3
	//查询数据
	err := db.Select(&person, sqlStr, user_id)
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}

	fmt.Println("select succ:", person)
}
```

## Update操作
```Go
// 更新数据
func updateDemo() {
	sqlStr := "update person set username=? where user_id=?"
	username := "进击的诸葛青"
	user_id := 2
	res, err := db.Exec(sqlStr, username, user_id)
	if err != nil {
		fmt.Printf("exec failed, err:%v\n", err)
		return
	}
	row, err := res.RowsAffected() //影响的行数
	if err != nil {
		fmt.Println("rows failed, ", err)
	}
	fmt.Println("update succ:", row)
}
```

## Delete操作

```GO
// 删除数据
func deleteDemo() {
	sqlStr := "delete from person where user_id=?"
	user_id := 2
	res, err := db.Exec(sqlStr, user_id)
	if err != nil {
		fmt.Println("exec failed, ", err)
		return
	}
	row, err := res.RowsAffected() //影响行数
	if err != nil {
		fmt.Println("rows failed, ", err)
	}

	fmt.Println("delete succ: ", row)

}
```
## NamedExec
> db.NamedExec方法用来绑定SQL语句与结构体或map中的同名字段，执行增删改操作
```Go
func nameExecDemo() {
	_,err := db.NamedExec("insert into person(username, sex, email)values(:username, :sex, :email)",
		Person{
		Username:"HTTP权威指南",
		Sex: "",
		Email: "",
		})
	if err != nil {
		fmt.Println(err)
		return
	}

	fmt.Println("执行成功")
}
```

## NamedQuery
> db.NamedQuery方法用来绑定SQL语句与结构体或map中的同名字段，进行查询操作
```

```

## 预处理
> * 1.优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。
> * 2.避免SQL注入问题

```Go
func prepareDemo() {
	sqlStr := "insert into person(username, sex, email)values(?, ?, ?)"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Println("error: prepared statement failed: ", err)
		return 
	}
	fmt.Println("prepare succeeded")
	stmt.Exec("记忆",nil, nil)
	stmt.Exec("redis设计与实现", nil, nil)
	stmt.Exec("mysql；innodb存储引擎", nil, nil)
	return
}
```



## 事务
`mysql事务特性`
>   1. 原子性（Atomicity）：一个事务（transaction）中的所有操作，`要么全部完成，要么全部不完成`，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
>    2. 一致性（Consistency）：事务按照预期生效，数据的状态是预期的状态。
>    3. 隔离性（Isolation）：多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
>    4. 持久性（Durability）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

`Db.Begin() 开始事务`
`Db.Commit()  提交事务`
`Db.Rollback()  回滚事务`
```Go
// 事务操作示例
func transactionDemo() {
	tx, err := db.Begin() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "insert into person(username, sex, email) values(?,?,?)"
	res1, err := tx.Exec(sqlStr1, "无限可能", "无", nil)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	affRow1, err := res1.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec res1.RowsAffected() failed, err:%v\n", err)
		return
	}
	//更新数据
	sqlStr2 := "Update person set sex=? where user_id=?"
	res2, err := tx.Exec(sqlStr2, "女", 3)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
	affRow2, err := res2.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec res1.RowsAffected() failed, err:%v\n", err)
		return
	}

	fmt.Println(affRow1, affRow2)
	if affRow1 == 1 && affRow2 == 1 {
		fmt.Println("事务提交啦...")
		tx.Commit() // 提交事务
	} else {
		tx.Rollback()
		fmt.Println("事务回滚啦...")
	}

	fmt.Println("exec trans success!")
}
```
