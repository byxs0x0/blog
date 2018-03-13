---
title: php函数漏洞小结
date: 2018-03-06 15:39:08
categories: Web
tags:
 - phpsec
 - ctf
---

记录一下自己学习中遇见的php函数漏洞
<!-- more -->
<br>
### ereg()
`int ereg ( string \$pattern , string \$string [, array &\$regs ] )`
用 pattern 的规则来解析比对字符串 string，比对结果返回的值放在数组参数 regs 之中。若省略参数 regs，则只是单纯地比对，找到则返回值为 true。
**以下特性eregi()函数也拥有**

**ereg()的%00截断**
```php
例子与用法如下
var_dump(@ereg("^[a-zA-Z0-9]+$", $_GET['id']));
id参数值       返回结果
123abc          true
123%00*&^%      true
123&^^%$        false

-----------------------------------------------------
这边有以下两点细节
1. %00不仅能作为$string中的截断，也能作为$pattern的截断
2. %00不是字符串的%00，必须是GET或者POST请求传入的参数为123%00*&^,注意url编码
```


另外%00怎么理解，从另一篇文章中领悟了一些，[文章链接](http://www.mottoin.com/18439.html)

>%00截断时的下面两种情况：
1.在url中加入%00，如http://xxoo/shell.php%00.jpg
url中的%00（形如%xx）,web server会把它当作十六进制处理，然后将该十六进制数据hex（00）“翻译”成统一的ascii码值“NUL（null）”，实现了截断。
2.在burpsuite中用burp自带的十六进制编辑工具将”shell.php .jpg”(中间有空格)中的空格由20改成00
所以在用python（或其它语言）中，要想“写出”截断符（null），也就是要写出ascii码值的NUL（null），也就是要由hex（十六进制）下的00变成ascii码值，这是语言和解释器以及计算机之间的关系


<br>
### extract()
`int extract ( array &\$array [, int \$flags = EXTR_OVERWRITE [, string \$prefix = NULL ]] )`
从数组中将变量导入到当前的符号表
```php
会造成变量覆盖，如果其他为为默认参数
$a = "test"; // 原变量值
$my_array = array("a" => "Cat","b" => "Dog", "c" => "Horse");
extract($my_array); // 变量覆盖
echo "\$a = $a; \$b = $b; \$c = $c";
// $a = Cat; $b = Dog; $c = Horse
```
<br>

### parse_str()
void parse_str ( string \$str [, array &\$arr ] )
将字符串解析成多个变量
```php
变量覆盖
// index.php?id=11
$id = 2;
parse_str($_SERVER['QUERY_STRING']); // id=11
echo $id; // 11
```


<br>
<br>
<br>

之后有遇见的在继续补充......
<br>
