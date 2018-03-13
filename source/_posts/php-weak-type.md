---
title: php弱类型小结
date: 2018-03-06 14:39:35
categories: Web
tags:
 - phpsec
 - ctf
---

php属于弱类型语言，在定义变量时，不需要声明变量类型，并且能自由的转换类型。
>然而，php内核的开发者原本是想让程序员借由这种不需要声明的体系，更加高效的开发，所以在几乎所有内置函数以及基本结构中使用了很多松散的比较和转换，防止程序中的变量因为程序员的不规范而频繁的报错，然而这却带来了安全问题。


<!-- more -->


<br>
## 类型比较
### 数字和字符
```php
数字和字符比较时，字符会被自动转换为数字类型

转换规则：
取从开头到碰见的第一个非数字字符之前为止作为转换结果，不存在则返回0
var_dump(0 == 'abcd');         // true
var_dump(123 == 'abcd');       // false
var_dump(123 == 'abcd123');    // false
var_dump(123 == 'ab123cd');    // false
var_dump(123 == 'ab123c44d');  // false
var_dump(123 == '123abcd');    // true
var_dump(123 == '00123abcd');  // true
var_dump(123 == '123a456bcd'); // true
var_dump(123 == '  123abcd');  // true
```

<br>
### Hash比较
```php
比较运算中，遇到0e\d+的字符串时，会将这种字符串解析为科学计数法。
'0e123' > 0*10^123 == 0
var_dump('0e123' == 0); // true
var_dump('0e123' == '0e456'); // true
var_dump('0e123' == '0e123abc'); // false
```
[0e开头](http://www.cnblogs.com/Primzahl/p/6018158.html)

<br>

### 16进制数进行比较
```php
16进制数会自动转换为整数类型
var_dump(0xff); // 255
var_dump('0xff'); // '0xff'
var_dump(intval('0xff')); // 0

0x开头（符合16进制数格式）的字符串与其他类型比较时，会转换为同一进制比较
var_dump('0xff' == 255);   // true
var_dump('0xff' == '255'); // true
var_dump(0x0 == 'aaaa');   // true
```
<br>

## 内置函数的松散性
### intval()
获取变量的整数值
```php
取从开头到碰见的第一个非数字字符之前为止作为转换结果，不存在则返回0
var_dump(intval('abc'));   // 0
var_dump(intval('1abc'));  // 1
var_dump(intval('abc1'));  // 0
var_dump(intval('ab1c'));  // 0
var_dump(intval('a1b1c')); // 0
```
<br>

### switch
switch在实验中得到几个特性：
1. case是int类型时，switch会自动将传入的参数转换为int类型
2. 如果case中没有写break，则会从从判断条件开始的case向下执行完，或者**遇见break停止**，例如下列代码：
```php
$temp ="1a";
switch ($temp) {
	case 0:
		echo "this is 0"."<br />";
	case 1:
		echo "this is 1"."<br />";
	case 2:
	    echo "this is 2"."<br />";
	    break;
	case 3:
	    echo "this is 3"."<br />";
	}
	//result:
	//this is 1
	//this is 2
```
<br>

### md5()
对字符串进行MD5加密
**以下特性sha1()也具有**
```php
MD5函数参数传入一个数组，函数无法处理数组，会返回null，并报一个Warning错误
Warning: md5() expects parameter 1 to be string, array given in xxx
$arr1 = array('1');
$arr2 = array('2');
var_dump(md5($arr1)); // null
var_dump(md5($arr1) == md5($arr2)) // true

// url?hello[]=123
$hello = $_GET["hello"];
var_dump($hello); // array(1) { [0]=> string(0) "" }
var_dump(md5($hello)); // null
```
<br>
### strcmp()
比较两个字符串
实际上是将两个变量转换成ascii 然后做数学减法，根据差值来返回一个int的差值(-1、0、1)。
**以下特性strpos()也具有**
```php
当传入参数为数组的时候，则返回为null
$arr = array('a');
var_dump(strcmp($arr,'a')); // null
我们可以让这个函数出错来绕过函数的检查
```

<br>

### in_array()
`bool in_array ( mixed \$needle,array \$haystack[, bool \$strict = FALSE ] )`
检查数组中是否存在某个值
当strict为True时，才严格检测即===
```php
$array = array(0, 1, '2');
var_dump(in_array("abc", $array)); // true
var_dump(in_array("1cc", $array)); // true
var_dump(in_array("2cc", $array)); // false
传入的字符串遇见数组中int类型进行比较时，传入的字符串会强转为int类型
```
<Br />
### is_numeric()
检测变量是否为数字或数字字符串
```php
var_dump(is_numeric(123));       # true
var_dump(is_numeric('123'));     # true
var_dump(is_numeric('123abc'));  # false
对16进制也能检测
var_dump(is_numeric(0x123));     # true
var_dump(is_numeric('0x123'));   # true
```

[实例](http://www.freebuf.com/articles/web/55075.html)


<br>

## 总结
在看到过的文章中一个非常好的总结
**在所有php认为是int的地方输入string，都会被强制转换**

<br>

## 参考
[PHP弱类型安全问题总结](https://blog.spoock.com/2016/06/25/weakly-typed-security/)
[浅谈PHP弱类型安全](http://wooyun.jozxing.cc/static/drops/tips-4483.html)
