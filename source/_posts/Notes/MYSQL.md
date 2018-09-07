---
title: MYSQL备忘录
date: 2018-07-05 10:44:55
categories: Notes
tags:
description:
---

记录一下学习MYSQL的要点
<!-- more -->

MySQL是一个开源的关系型数据库管理系统，分为社区版和企业版。

<br/>
# MySQL安装目录结构
大致可以分为启动文件、配置文件、数据库文件、命令文件。
 - bin目录：用于放置一些可执行文件，比如mysql.exe、mysqld.exe、mysqlshow.exe等。 
 - data目录：用于放置一些日志文件以及数据库。 
 - include目录：用于放置一些头文件，比如mysql.h、mysqld_ername.h等。 
 - lib目录：用于放置一系列的库文件。 
 - share目录：用于存放字符集、语言等信息。 
 - my.ini目录：是MySQL数据库中使用的配置文件。 
 - my-huge.ini文件：适合超大型数据库的配置文件。 
 - my-largte.ini文件：适合大型数据库的配置文件。 
 - my-medium.ini文件：适合中型数据库的配置文件。 
 - my-small.ini文件：适合小型数据库的配置文件。 
 - my-template.ini文件：是配置文件的模板，MySQL配置向导将该配置文件中选择项写入到my.ini文件。 
 - my-innodb-heavy-4G.ini文件：表示该配置文件只对于InnoDB存储引擎有效，而且服务器的内存不能小于4GB。
 - 注意:上述的7个配置文件，其中my.ini是MySQL正在使用的配置文件，该文件是一定会被读取的，其他的配置文件都是适合不同数据库的配置文件的模板，会在某些特殊情况下被读取，如果没有特殊需求，只需要配置my.ini文件即可。

<br/>
# MySQL配置文件
Window和Linux配置文件名称一般不同
 - Window `my.ini`
 - Linux `my.cnf`
配置文件中会有两大块内容
 - MySQL客户端配置
   - `[client]`
     - `port=3306`就是使用客户端连接MySQL服务器的默认端口，如果不是3306，可以用`-P`参数指定
   - `[mysql]`
     - `default-character-set=utf8`指定了默认编码格式
 - MySQL服务端配置
   - `[mysqld]`

<br/>
# MySQL语句规范
 - 关键字和函数名称全部大写
 - 数据库名称、表名称、字段名称全部小写
 - SQL语句必须以分号结尾

<br/>
# MySQL命令
 - 花括号代表必选项
 - 竖线代表多个选一个
 - 中括号代表可选项


## 提示符
 - `mysql -uroot -proot --prompt 提示符`
 - `prompt 提示符`
   - `\D` 完整日期
   - `\d` 当前数据库
   - `\h` 当前主机地址
   - `\u` 当前用户

## MySQL常用命令
 - `SELECT VERSION()`
 - `SELECT NOW()`
 - `SELECT USER()`

## 数据库操作

### 创建数据库
`CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [DEFAULT] CHARACTER SET [=] charset_name`

### 修改数据库
`ALTER {DATABASE | SCHEMA} [db_name] [DEFAULT] CHARACTER SET [=] charset_name`

### 删除数据库
`DROP {DATABASE | SCHEMA} [IF EXISTS] db_name`

### 打开数据库
`USE db_name`

## 数据表操作

### 创建数据表
```sql
-- 语法
CREATE TABLE [IF NOT EXISTS] table_name(
    column_name data_type,
    ...
)

-- 实例
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_id` INT UNSIGNED AUTO_INCREMENT,
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

#### 约束
 - 约束保证数据的完整性和一致性
 - 约束分为表级约束和列级约束
 - 约束类型包括
    - `NOT NULL`(非空约束)
      - 不能为NULL，否则会报错
    - `PRIMARY KEY`(主键约束)
      - 代表主键约束，保证记录的唯一性
      - 每张表只能存在一个主键
      - 主键自动为 NOT NULL
      - 主键不一定和AUTO_INCREMENT使用
    - `UNIQUE KEY`(唯一约束)
      - 代表唯一约束，保证记录的唯一性
      - 字段可以为空值(NULL)
      - 每张数据表可以存在多个唯一约束
    - `DEFAULT`(默认约束)
      - 默认约束，当插入记录时，会给没有值的字段自动赋予默认值
    - `FOREIGN KEY`(外键约束)
      - 保证了数据一致性和完整性
      - 前提条件比较坎坷
 
 - `AUTO_INCREMENT`
  - 必须和主键一起使用
  - 默认情况下起始值为1，自增长值为1
  - 数据类型可以是整数也可以是浮点数，但是浮点数时，小数位必须为0
 - `UNSIGNED`
   - 整数数据类型无符号位(也就是无负数)
 - `ENGINE`
   - 设置存储引擎
 - `CHARSET=utf8`
   - 设置字符集格式



### 查看数据表
`SHOW TABLES [FROM db_name] [LIKE 'pattern' | WHERE expr]`

### 添加单列
`ALTER TABLE tbl_name ADD [COLUMN] col_name column_definition [FIRST | AFTER col_name]`



### 删除数据表
`DROP TABLE tb_name`

### 查看数据表结构
`SHOW COLUMNS FROM tbl_name`
`DESC tbl_name`


## 数据操作
### 数据记录查看
```sql
SELECT 
    [ALL | DISTINCT | DISTINCTROW ] 
      [HIGH_PRIORITY] 
      [STRAIGHT_JOIN] 
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT] 
      [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS] 
    select_expr [, select_expr ...] 
    [FROM table_references 
    [WHERE where_condition] 
    [GROUP BY {col_name | expr | position} 
      [ASC | DESC], ... [WITH ROLLUP]] 
    [HAVING where_condition] 
    [ORDER BY {col_name | expr | position} 
      [ASC | DESC], ...] 
    [LIMIT {[offset,] row_count | row_count OFFSET offset}] 
    [PROCEDURE procedure_name(argument_list)] 
    [INTO OUTFILE 'file_name' export_options 
      | INTO DUMPFILE 'file_name' 
      | INTO var_name [, var_name]] 
    [FOR UPDATE | LOCK IN SHARE MODE]]
```
 - `select_expr`查询表达式
   - 每一个表达式表示想要的一列，必须有至少一个。
   - 多个列之间以英文逗号分隔。
   - 星号(*)表示所有列。tbl_name.*可以表示命名表的所有列。
   - 表达式可以使用[AS] alias_name 为其赋予别名。
   - 别名可用于GROUP BY， ORDER BY 或 HAVING子句。
 - `GROUP BY`对查询结果集进行分组
   - 例如按照相同年龄的分组`SELECT COUNT(*),age FROM users GROUP BY age`
 - `HAVING`分组后的过滤条件
   - HAVING后面必须为聚合函数或者是SELECT的字段

### 插入数据
如果给默认增长的字段赋值，可以输入`NULL`或`DEFAULT`
INSERT有多种方式，各有不同:
 - `INSERT [INTO] tbl_name [(col_name,...)] {VALUES | VALUE} ({expr | DEFAULT},...),(...),...`
   - 可以插入多条语句
 - `INSERT [INTO] tbl_name SET col_name={expr | DEFAULT},...`
   - 可以使用子查询
   - 只能单条插入
 - `INSERT [INTO] tbl_name [(col_name,...)] SELECT ...`
   - 将查询结果写入到INSERT


### 修改数据
`UPDATE [LOW_PRIORITY] [IGNORE] tb_name SET col_name={expr | DEFAULT}[,col_name={expr | DEFAULT}]... [WHERE where_condition]`
注意有个名词 **多表更新**

### 删除数据
`DELETE FROM tb_name [WHERE where_condition]`


## 查看警告信息
`SHOW WARNINGS` 

## 子查询
在SQL语句内的SELECT子句
 - 子查询必须始终出现在圆括号内。
 - 子查询可以包含多个关键字或条件。
 - 子查询的外层查询可以是`SELECT INSERT UPDATE SET DO`。
 - 子查询返回值可以是标量、一行、一列或子查询。

当子查询返回多行记录，可以使用`[NOT] IN、[NOT] EXISTS`来做条件筛选
同样的还有`ANY SOME ALL`关键字
![子查询表](2018-07-06-08-56-47.png)


## 连接
### 连接类型
 - `INNER JOIN` 内连接
   - 仅显示符合连接条件的语句
   - 在MYSQL中，`JOIN` `CROSS JOIN` `INNER JOIN`是等价的
 - `LEFT [OUTER] JOIN`左外连接
   - 显示左表全部和右表中符合连接条件的，不符合会显示为NULL
 - `RIGHT [OUTER] JOIN`右外连接
   - 显示右表全部和左表符合连接条件的，不符合会显示为NULL
### 连接条件
使用`ON`关键字来设定连接条件，也可以使用`WHERE`来代替。
通常使用`ON`关键字来设置连接条件，
使用`WHERE`关键字进行结果集记录和过滤。


<br/><br/>
# 数据类型
数据类型是指列、存储过程参数、表达式和局部变量的数据特征，它决定了数据的存储格式，代表了不同的信息类型。

## 整型

![整型类型表](2018-07-05-14-52-43.png)

## 浮点型

![浮点类型表](2018-07-05-14-55-10.png)

## 日期时间型

![日期时间类型表](2018-07-05-14-59-11.png)

## 字符型

![字符类型表](2018-07-05-15-00-46.png)


# 函数与运算符

## 字符函数
![字符函数](2018-07-06-14-33-16.png)
![字符函数2](2018-07-06-14-33-38.png)



## 数值运算符与函数
![数值运算符与函数](2018-07-06-14-34-13.png)



## 比较运算符和函数
![比较运算符和函数](2018-07-06-14-34-37.png)


## 日期时间函数
![日期时间函数](2018-07-06-14-31-04.png)


## 信息函数
![信息函数](2018-07-06-14-31-41.png)



## 聚合函数
![聚合函数](2018-07-06-14-32-18.png)


## 加密函数
![加密函数](2018-07-06-14-32-54.png)


<br/><br/>
# 自定义函数
用户自定义函数(user-defined function,UDF)是一种对MySQL扩展的途径，其用法与内置函数相同。

自定义函数的两个必要条件：
 - 参数
 - 返回值

自定义函数语法
```sql
CREATE FUNCTION funcion_name
RETURNS
{STRING|INTEGER|GEAL|DECIMAL}
routine_body -- 函数体
```

关于函数体：
 - 函数体由合法的SQL语句构成
 - 函数体可以是简单的SELECT或INSERT语句
 - 函数体如果为复合结构则使用BEGIN...END语句
 - 符合结构可以包含声明，循环，控制结构

定义一个简单的函数，将SELECT NOW()格式改变
```SQL
$ SELECT NOW();
+---------------------+
| NOW()               |
+---------------------+
| 2018-07-06 16:20:55 |
+---------------------+
1 row in set (0.00 sec)

$ CREATE FUNCTION NOWS() RETURNS VARCHAR(30)
    -> RETURN DATE_FORMAT(NOW(),'%Y %m %d %H:%i:%s');
Query OK, 0 rows affected (0.00 sec)

$ SELECT NOWS();
+---------------------+
| NOWS()              |
+---------------------+
| 2018 07 06 16:22:03 |
+---------------------+
1 row in set (0.00 sec)
```

定一个带有参数的函数，计算两个数的平均值
```SQL
$ CREATE FUNCTION AVGS(num1 SMALLINT UNSIGNED,num2 SMALLINT UNSIGNED)
    -> RETURNS FLOAT(10,2)
    -> RETURN (num1 + num2) / 2;
Query OK, 0 rows affected (0.00 sec)

$ SELECT AVGS(100,2000);
+----------------+
| AVGS(100,2000) |
+----------------+
|        1050.00 |
+----------------+
1 row in set (0.00 sec)
```
 
定一个具有复杂结构的函数，添加一个用户并返回用户ID
`DELIMITER`的作用是更改结束符号，因为写在函数体的语句需要使用到`;`来结束
```sql
$ DELIMITER $$
$ CREATE FUNCTION ADDUSER(username VARCHAR(50))
    -> RETURNS INT UNSIGNED
    -> BEGIN
    -> INSERT users(username,password) VALUES(username,'123123');
    -> RETURN LAST_INSERT_ID();
    -> END
    -> $$
Query OK, 0 rows affected (0.00 sec)

$ SELECT ADDUSER('hahah');
+------------------+
| ADDUSER('hahah') |
+------------------+
|               43 |
+------------------+
1 row in set (0.00 sec)

$ SELECT * FROM users;
+----+----------------------+------------+
| id | username             | password   |
+----+----------------------+------------+
|  1 | Hello                | World      |
|  2 | Angelina             | I-kill-you |
|  3 | Dummy                | p@ssword   |
|  4 | secure               | crappy     |
|  5 | stupid               | stupidity  |
|  6 | superman             | genious    |
|  7 | batman               | mob!le     |
|  8 | admin                | admin      |
|  9 | admin1               | admin1     |
| 10 | admin2               | admin2     |
| 11 | admin3               | admin3     |
| 12 | dhakkan              | dumbo      |
| 14 | admin4               | admin4     |
| 15 | Dumb                 | cccc       |
| 43 | hahah                | 123123     |
+----+----------------------+------------+
15 rows in set (0.00 sec)
```

<br/><br/>
# 存储过程
存储过程(Stored Procedure)：
　　一组可编程的函数，是为了完成特定功能的SQL语句集，经编译创建并保存在数据库中，用户可通过指定存储过程的名字并给定参数(需要时)来调用执行。

存储过程和函数的区别：
 - 函数只能通过return语句返回单个值或者表对象。而存储过程不允许执行return，但是通过out参数返回多个值。
 - 函数是可以嵌入在sql中使用的,可以在select中调用，而存储过程不行。
 - 函数限制比较多，比如不能用临时表，只能用表变量．还有一些函数都不可用等等．而存储过程的限制相对就比较少
 - 一般来说，存储过程实现的功能要复杂一点，而函数的实现的功能针对性比较强。
 - 当存储过程和函数被执行的时候，SQL Manager会到procedure cache中去取相应的查询语句，如果在procedure cache里没有相应的查询语句，SQL Manager就会对存储过程和函数进行编译。
 - Procedure cache中保存的是执行计划 (execution plan) ，当编译好之后就执行procedure cache中的execution plan，之后SQL SERVER会根据每个execution plan的实际情况来考虑是否要在cache中保存这个plan，评判的标准一个是这个execution plan可能被使用的频率；其次是生成这个plan的代价，也就是编译的耗时。保存在cache中的plan在下次执行时就不用再编译了。

[具体可以看这篇文章](http://www.cnblogs.com/geaozhang/p/6797357.html)

![MYSQL命令执行流程](2018-07-06-16-41-45.png)


<br/><br/>
# 存储引擎
MySQL可以将数据以不同的技术存储在文件(内存)中，这种技术就被称为存储引擎。
每一种存储引擎使用不同的存储机制、索引技术、锁定水平，最终提供广泛且不同的功能。
存储引擎分为：
 - `MyISAM`
 - `InnoDB`
 - `Memory`
 - `Archive`
 - `CSV`

![存储引擎](2018-07-06-17-07-52.png)

