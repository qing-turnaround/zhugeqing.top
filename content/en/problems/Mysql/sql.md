---
date: 2021-11-24
description: "查询犹如算法"
image: "images/recommend_site/xingyouji.jpg"
title: "Sql题目"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- Sql
series:
- 
---
1. [第二高的薪水](#第二高的薪水)

## 第二高的薪水
题目地址:[入口](https://leetcode-cn.com/problems/second-highest-salary/)

* 题目：编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。
```Mysql 
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```
例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。

```Mysql
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```


* 解法：

`使用distinct来过滤相同的salary，使用order by进行排序，再加上DESC使其递减，用limit n表示返回前n条数据，offset n表示跳过n条语句，（limit 1 offset 1表示跳过前一条数据，再返回前一条数据），由于还需要返回null的情况，所以将其作为临时表`
```Mysql
SELECT
    (SELECT DISTINCT
            Salary
        FROM
            Employee
        ORDER BY Salary DESC
        LIMIT 1 OFFSET 1) AS SecondHighestSalary
;
```

`ifnull函数来判断表达式返回值是否为null，若为null，则返回括号后面的值，否则返回表达式的结果`
```Mysql
select 
ifnull(
    (select distinct salary 
    from Employee 
    ORDER BY salary DESC
    limit 1,1), null)
   as SecondHighestSalary
```
