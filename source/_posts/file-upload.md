---
title: 文件上传漏洞总结
date: 2018-02-19 16:54:18
tags:
 - phpsec
---

# 概述
大部分站点都具有文件上传功能，例如头像更改，文章编辑，附件上传等等。如果上传没有经过合理严谨的验证，或者服务器没有经过安全的配置，都可能导致文件上传漏洞。而文件上传漏洞会导致网站被控制甚至服务器沦陷。
[靶机代码在这](https://github.com/byxs0x0/php-file-upload)
<br>
# 文件上传验证流程
在文件上传功能一般会在前端做后缀名验证和对HTTP报文中几个关键点做验证。
 - 客户端JavaScript验证
 - 服务端目录路径检测
 - 服务端MIME类型验证
 - 服务端文件扩展名验证
 - 服务器文件内容验证

<br>
## 客户端验证
通常是用JavaScript代码来检测扩展名是否合法，来达到验证。

**示例代码**
```javascript
function checkUpload(fileobj){
  var fileArr = fileobj.value.split("."); //对文件名进行处理
  var ext = fileArr[fileArr.length-1]; //得到文件扩展名
  if(ext!='gif') //验证扩展名
  {
    alert("Only upload GIF images.");
    fileobj.value = ""; //清除数据
  }
}
<form action="upload.php" method="post" enctype="multipart/form-data" >
  <input type="file" name="fupload" onchange="checkUpload(this)" id="file" />
  <input type="submit" name="submit" value="upload!" />
</form>
```

**绕过方法**
 - 直接发送请求包
 通过Burp抓到正常上传的请求报文后，修改报文的内容，在直接通过Burp发送，便跳过了网页中JS的验证过程。
 - 修改JavaScript
 去修改其中关键的检测函数，或者直接通过`firebug`或`noscript`插件禁用JavaScript。

<br>
## 服务端验证

### 目录路径检测
其实就是开发人员在写代码时把`文件保存的路径`也放在了请求中，成为了用户的可控变量。

**示例代码**
```php
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
    $target_path = $_REQUEST["path"].$file_name; //存储路径与名称
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

**利用姿势**
 - 利用%00截断，在代码验证的时候是取`.gif`来进行，但是最后保存在本地时，%00会截断文件名，导致最终保存结果为`uploads/php.php`
![ 00-php](00-php.png)
 - 在asp中可以结合IIS解析漏洞，巧妙构造出文件名。例如
   - uploads.asp/  (uploads.asp/shell.jpg)
   - uploads/.asp/ (uploads/.asp/shell.jpg)
   - uploads/shell.asp; (uploads/shell.asp;shell.jpg)

<br>
### MIME类型检测
检测请求报文中`Content-Type`的值是否在允许的范围中。

**示例代码**
```php
if(isset($_POST["submit"])){
    // 检测Content-type
    if($_FILES['fupload']['type'] != "image/gif")
    {
        exit("Only upload GIF images.");
    }
    // 构建基本参数
    $file_name = $_FILES['fupload']['name']; // 文件名
    $file_ext  = substr($file_name, strrpos($file_name,'.') + 1); //文件后缀
    $file_tmp  = $_FILES['fupload']['tmp_name']; //临时文件
    $target_path = "uploads\\".md5(uniqid(rand())).".".$file_ext; //存储路径与名称
    // 移动临时文件
    if(move_uploaded_file($file_tmp, $target_path)){
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
    else{
        echo '<pre>upload error</pre>';
    }
}
```

**绕过方法**
利用Burp抓包，将报文中的Content-Type改成允许的类型
 - `Content-Type: image/gif`
 - `Content-Type: image/jpg`
 - `Content-Type: image/png`

<br>
### 文件扩展名检测
检测文件的扩展名是否在黑名单，或者白名单中。
#### 黑名单
禁止危险文件类型的上传，安全系数比较差。

**示例代码**
```php
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
    $black_ext = explode("|", "asp|asa|cer|cdx|aspx|ashx|ascx|asax|php|php2|php3|php4|php5|asis|htaccess|htm|html|shtml|pwml|phtml|phtm|js|jsp|vbs|asis|sh|reg|cgi|exe|dll|com|bat|pl|cfc|cfm|ini"); // 转换为数组
    if(in_array($file_ext,$black_ext))
    {
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
**绕过方法**
 - 后缀名大小写，例如pHp
 - 寻找黑名单中没有被禁止的文件类型
 以下文件同样会被解析
   - php|php2|php3|php4|php5  (好像只能基于debian和ubuntu的apt-get安装,否则是不存在该类型漏洞)
   - asp|aspx|asa|cer
   - exe|exee

 从别人那copy下来的黑名单列表
 - asp|asa|cer|cdx|aspx|ashx|ascx|asax
 - php|php2|php3|php4|php5|asis|htaccess
 - htm|html|shtml|pwml|phtml|phtm|js|jsp
 - vbs|asis|sh|reg|cgi|exe|dll|com|bat|pl|cfc|cfm|ini


#### 白名单
只允许上传指定文件类型。

**示例代码**
```php
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
**绕过方法**
从操作系统特性和服务器解析漏洞或其他姿势来思考。

<br>
### 文件内容检测
图片格式往往不是根据文件后缀名去做判断的。文件头是文件开头的一段二进制，不同的图片类型，文件头是不同的。文件头又称文件幻数。

#### 文件头检测
**常见文件幻数**
 - JPG：FF D8 FF E0 00 10 4A 46 49 46
 - GIF：47 49 46 38 39 61 (GIF89a)
 - PNG：89 50 4E 47

**示例代码**
```php
// 检测文件头
$file_info = getimagesize($file_tmp); //函数会返回图片的类型以及宽高
if($file_info["mime"] != "image/gif"){ // 是否是GIF
    exit("Only upload GIF images.");
}

这边有个有趣的现象
开头是文本的 GIF(完整应该是GIF89a) 也能通过以上代码的检测
所以以上只是简单的检测例子

exif_imagetype()也能进行检测，但是需要对php.ini进行设置
在Linux上：extension=exif.so
在Windows上：extension=php_exif.dll
Windows的注意事项：
Windows用户必须在php.ini中启用php_mbstring.dll和php_exif.dll DLL。必须在php_exif.dll DLL之前加载php_mbstring.dll DLL，以便相应地调整您的php.ini。
```
**绕过方法**
在php代码前构建正常文件幻数。或者在正常的图片后面加上php代码。
#### 文件加载检测(待续)
在其他的总结中都提到`图像渲染测试`和`二次渲染`，没有实验，不过大致说一下自己的理解

**图像渲染测试**
图像渲染测试是直接用代码来测试图像是否能呈现，如果是自己伪造的文件头或者篡改图片的内容，都可能让图片本身不能正常显示。
那么其中的绕过方法，其实是在加入代码的同时不破坏图片。

**二次渲染**
>将你上传的文件中属于图片部分的数据抓取出来，再使用自己的API或者函数重新将这张图片生成出来保存在服务端

我理解的二次渲染过程，比如`加上水印`或者`生成缩略图`这样的过程。

**注意**
如果过滤了内容中<?php，可是使用其他标记
 - `<?php ?>`
 - `<? ?>` (开启short_open_tag)
 - `<% %>` (开启asp_tags)
 - `<script language="php"></script>`

<br><br>


# WAF绕过
参照他[我的WafBypass之道（upload篇）](https://paper.seebug.org/219/)
自己还没做过实验！！！
<br><br>


# 利用操作系统特性
## window特殊字符
利用window对于文件和文件名的限制，以下字符放在结尾时，不符合操作系统的命名规范，在最后生成文件时，字符会被自动去除。

|    上传文件名     | 服务器文件名 |                                                    说明                                                    |
|:----------------- |:------------ | ---------------------------------------------------------------------------------------------------------- |
| file.php[空格]    | file.php     |                                                                                                            |
| file.php[.]       | file.php     |                                             无论多少个.都可以                                              |
| file.php[%80-%99] | file.php     | Burp抓包，在文件名结尾输`%80`，`ctrl+shift+u`进行`url-decode`，或者增加一个空格,再在在Hex视图为把20修改为80 |

## NTFS ADS特性
不是特别理解，记录一下根据结论，自己实验的结果
语法格式：`<filename>:<stream name>:<stream type>`
>The default data stream has no name. That is, the fully qualified name for the default stream for a file called "sample.txt" is "sample.txt::$DATA" since "sample.txt" is the name of the file and "$DATA" is the stream type.

|      上传文件名       | 服务器生成的文件名 | 内容 |
|:--------------------- |:------------------ | ---- |
| file.php:jpg          | file.php           |  空  |
| file.php::$DATA       | file.php           | 实际 |
| file.php::$DATA...... | file.php           | 实际 |

还有一种姿势，利用`file.php:jpg`生成空php文件，在利用`file.<<<`来覆盖。
参考这篇例子[当php懈垢windows通用上传缺陷](https://ctolib.com/topics-88860.html)
<br><br>
# 利用服务器解析漏洞
服务器解析漏洞是在某种特定的场合下，一些文件被iis、apache、nginx等解析为脚本文件并执行产生的漏洞。

## IIS6.0解析漏洞

### 目录解析
目录名为`.asp`、`.asa`、`.cer`，则目录下的所有文件都会被作为ASP解析。
`url/test.asp/shell.jpg`会被当作asp脚本运行。

### 文件解析
文件名中如果包含`.asp;`、`.asa;`、`.cer;`则优先使用asp解析。
`url/test.asp;shell.jpg`会被当作asp脚本运行。

### 文件类型解析
iis6.0中`.asa`，`.cer`，`.cdx`都会被作为asp文件执行。
`url/shell.asa`会被作为asp文件执行。

## apache解析漏洞
apache解析文件规则是从右到左。例如`shel.php.gix.ccc`，apache会先识别ccc，ccc不被识别，则识别gix，以此类推，最后会被识别为php来运行。

**版本测试**
```
PHPWAMP  apache 2.4.29(Server API=Apache 2.0 Handler) [存在]
PHPStudy apache 2.4.23(Server API=Apache 2.4 Handler - Apache Lounge) [不存在]
```

## Nginx解析漏洞
PHP+nginx默认是以cgi的方式去运行，当用户配置不当，会导致任意文件被当作php去解析。
(IIS7、IIS7.5也具有，暂没实验)
**利用条件**
 - 以`FastCGI`运行
 - `cgi.fix_pathinfo=1`(全版本PHP默认为开启)

例如如果满足上述条件，当你访问`url/shell.jpg/shell.php`时，shell.jpg会被当作php去执行。
原理参考这篇文章[Nginx解析漏洞原理分析](http://byd.dropsec.xyz/2017/11/09/Nginx%E8%A7%A3%E6%9E%90%E6%BC%8F%E6%B4%9E%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/)
[Nginx 解析漏洞复现](https://github.com/vulhub/vulhub/tree/master/nginx_parsing_vulnerability)
[Fastcgi协议分析 && PHP-FPM未授权访问漏洞 && Exp编写](https://paper.seebug.org/289/)

## Nginx空字节代码执行漏洞
**版本影响**
 - `0.5.*, 0.6.*, 0.7 <= 0.7.65, 0.8 <= 0.8.37`

利用方式就是`url/shell.jpg%00.php`
由于没有环境实验，待续..

## Nginx文件名逻辑漏洞
很好的一个思路，具体的看[Nginx 文件名逻辑漏洞（CVE-2013-4547）](https://github.com/vulhub/vulhub/tree/master/nginx/CVE-2013-4547)

**版本影响**
 - `0.8.41 ~ 1.4.3 / 1.5.0 ~ 1.5.7`
简单说下利用方法
```
1.上传一个`shell.jpg `文件，注意最后为空格
2.访问`url/shell.jpg[0x20][0x00].php`
(两个中括号中的数字是用Burp在Hex界面中更改)
```
<br><br>
# 利用CMS、编辑器漏洞
 - 寻找CMS中文件上传的CVE
 - 看文件上传功能是否是编辑器提供，如果是寻找这个版本编辑器是否存在漏洞。

<br><br>
# 其他利用

## %00截断
当上传的文件名为`shell.php[0x00].jpg`([0x00]在Burp中Hex界面更改，不是字符串)。当代码验证后缀名时，会取`.jpg`。从而通过验证，但是将文件保存在指定目录时，0x00会将文件名阶段，从而保存的文件名实际为`shell.php`。
此利用手法好像只在asp程序中。(php5.2.17中没成功)

## apache配置不当
 - 在apache的配置文件中如果拥有`AddHandler php5-script .php`,则即使`shell.php.jpg`也会被当作php去执行。
 - 在apache的配置文件中如果拥有`AddType application/x-httpd-php .jpg`,则即使`shell.jpg`也会被当作php去执行。

## .htaccess文件攻击
>在apache里，这个文件作为一个配置文件，可以用来控制所在目录的访问权限以及解析设置。即是，可以通过设置可以将该目录下的所有文件作为php文件来解析，即可绕过

但是利用前提是需要先想办法去重写.htaccess文件(我也不知道，要好好研究),然后内容如
```
<FilesMatch "shell">
SetHandler application/x-httpd-php
</FilesMatch>
```
当你上传`shell.jpg`文件后，直接访问会被当作php去执行。

## 竞争上传
大致说一下思路，竞争上传其实就是利用上传中的空隙，例如[这篇文章中的例子](http://jdrops.dropsec.xyz/2017/07/17/%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/)中。
如果文件上传验证过程是，先将文件保存在本地，然后在去验证文件是否符合验证标准，如果不符合在删除。那么借着删除的空隙，在删除之前把脚本执行了，遍可以执行任意代码了。当然执行脚本必然是利用脚本去实现。
[还有一个类似的例子](https://chybeta.github.io/2017/08/22/XMAN%E5%A4%8F%E4%BB%A4%E8%90%A5-2017-babyweb-writeup/)

## 双文件上传
如果代码存在bug，能同时上传多个文件，但是校验时只对第一个上传的文件进行校验，便可以导致双文件绕过。


<br>
# 图片木马制作
window下cmd中执行`copy /b pic.jpg+shell.php`，将shell.php的内容加到pic.jpg结尾。并且图像合并必须使用二进制(/b)。
注：直接将代码放到结尾也一样，至少我的实验结果是这样。
<br>
# 防御
 - 限制图片目录的权限。(设置不可执行脚本)
 - 文件验证中使用白名单验证+文件名重命名等多种方式验证。
 - 对文件内容进行校验。
 - 对服务器对安全配置。

<br>
# 总结思路
以上都是从各种文章中总结实验下的内容，借助这个过程把心里的思路好好的理一下。感谢师傅们的经验。

 - 判断上传点是编辑器或CMS还是开发人员写的。
   - 如果是编辑器或者CMS漏洞，寻找POC利用。
 - 如果是开发人员写的，从文件上传验证流程去思考他代码验证的构成。
 - 从操作系统特性思考
 - 从服务器解析漏洞思考
 - 从其他姿势思考

<br>
# 参照
 - [文件上传漏洞（绕过姿势）](https://thief.one/2016/09/22/%E4%B8%8A%E4%BC%A0%E6%9C%A8%E9%A9%AC%E5%A7%BF%E5%8A%BF%E6%B1%87%E6%80%BB-%E6%AC%A2%E8%BF%8E%E8%A1%A5%E5%85%85/)
 - [文件上传绕过姿势总结](http://www.cnnetarmy.com/%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E7%BB%95%E8%BF%87%E5%A7%BF%E5%8A%BF%E6%80%BB%E7%BB%93/)
 - [文件上传总结](https://masterxsec.github.io/2017/04/26/%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%80%BB%E7%BB%93/)
 - [文件上传漏洞总结](http://jdrops.dropsec.xyz/2017/07/17/%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/)
 - [上传验证绕过全解析](http://www.cnblogs.com/cyjaysun/p/4439058.html)
 - [服务器解析漏洞](https://thief.one/2016/09/21/%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%A7%A3%E6%9E%90%E6%BC%8F%E6%B4%9E/)
 - [文件解析漏洞总结-Nginx](https://blog.werner.wiki/file-resolution-vulnerability-nginx/)
 - [解析漏洞总结](http://wps2015.org/drops/drops/%E8%A7%A3%E6%9E%90%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93.html)
