---
title: 关于HTTP重定向的多种方法
date: 2018-06-24 17:46:40
categories: Write Up
tags:
 - ctf
 - wp
description:
---

对几种HTTP重定向做一个自我理解的总结
<!-- more -->

>URL 重定向，也称为 URL 转发，是一种当实际资源，如单个页面、表单或者整个 Web 应用被迁移到新的 URL 下的时候，保持（原有）链接可用的技术。HTTP 协议提供了一种特殊形式的响应—— HTTP 重定向（HTTP redirects）来执行此类操作，该操作可以应用于多种多样的目标：网站维护期间的临时跳转，网站架构改变后为了保持外部链接继续可用的永久重定向，上传文件时的表示进度的页面，等等。

<br>
### HTTP重定向机制
HTTP重定向就是浏览器会根据服务器回送的响应报文来进行重定向。
![HTTP重定向图片](2018-06-27-16-42-07.png)

如上图，浏览器接受了服务器的301重定向响应后，会根据提供的新URL进行重定向。
而HTTP协议的重定向响应的状态码为3xx。不同类型的重定向状态码也包含了不同的含义。但是也可以分为3个类型：
 - 永久重定向
   - 表示原来的URL已经被废弃，例如在站点迁移需要设置301重定向告诉搜索引擎你的网站已经迁移。
   - 除了`301 Moved Permanently`常见之外，还有`308 Permanent Redirect`
 - 临时重定向
   - 有时候请求的资源无法从其标准地址访问，但是却可以从另外的地方访问。搜索引擎不会记录该新的、临时的链接。
   - `302 Found`状态码是最常见的，例如当页面产生报错，未在登录状态下进入需要登录权限的界面等等
   - 还有`303 see Other`与`307 emporary Redirect`
 - 特殊重定向
   - `304`（Not Modified，资源未被修改）是最常见的，它说明页面没有发生变化，浏览器会直接从本地缓存读取数据，搜索引擎遇见304，也会认为这个网页没有改动，不会分析其内容
   - 还有`300 Multiple Choice`

 更详细可以看[MDN-HTTP 的重定向](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Redirections)


<br>
#### 实例分析
在PHP中，我们可以使用`header()`函数来定制响应报文头来做实验

##### 301重定向
测试代码：
```
// 301
header( "HTTP/1.1 301 Moved Permanently" );
header( "Location: http://example.com/" );
```
响应：
![301](2018-06-26-11-34-44.png)
可以明显的看见浏览器接受了301响应后会根据`Location`字段提供的新URL进行重定向

##### 302重定向
测试代码：
```
// 302  不指定状态码，默认为302
header( "Location: http://example.com/" );
```
响应：
![302](2018-06-26-11-37-54.png)
与301差别就是状态码的差别，效果一致

##### Refresh页面刷新
`Refresh`字段规定浏览器延时刷新页面或跳转到指定URL中
测试代码：
```
// Refresh
header( "Refresh: 5" ); // 5秒一次刷新页面
// header( "Refresh: 0; url=http://example.com/" ); // 0秒后跳转到指定URL中
```
实例：
![Refresh](2018-06-26-12-02-25.png)
Refresh并不是重定向，但是它在某些实际效果中也能实现类型的功能

<br>
### HTML重定向机制
浏览器读取页面后，检测到该元素，会在`0`秒后跳转到`http://example.com/`中
```html
<head>
    <meta http-equiv="refresh" content="0; url=http://example.com/" />
</head>

注意跳转后会无法在浏览器按回退按钮回退
```
<br>
### JavsScript重定向机制
```javascript
<script type="text/javascript">
	window.location = "http://example.com/";
	window.location.href = "http://example.com/";
	self.location = "http://example.com/";
	top.location = "http://example.com/";
</script>
```

`JavaScript`的跳转方式非常多，可以参考[这篇文章](http://qianduandu.com/277.html)

<br>
### 服务端跳转
上面说的都是在客户端进行跳转，服务端跳转其实就是服务器去获得跳转后的页面数据，在交给客户端

