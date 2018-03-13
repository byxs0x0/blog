---
title: 关于PHP伪造IP或来源地址
date: 2018-02-06 16:29:46
categories: Web
tags:
 - phpsec
 - ctf
---

做了些关于伪造IP的CTF题目，看了些别人的文章，稍微自己总结一下

<!-- more -->
----------
### 实验环境
服务器操作系统：window
Client IP：`192.168.220.1`
Server IP：`192.168.220.141`
### HTTP请求报文
```HTTP
# 删除了一些不重要的请求头
GET /test.php HTTP/1.1
Host: 192.168.220.141
X-Forwarded-For: 172.16.10.10
CLIENT-IP: 100.100.100.100
Referer: www.google.com
```
### test.php

```php
echo $_SERVER["HTTP_CLIENT_IP"].'<br />'; // 100.100.100.100
echo $_SERVER["HTTP_X_FORWARDED_FOR"].'<br />'; //172.16.10.10
echo $_SERVER["REMOTE_ADDR"].'<br />'; // 192.168.220.1
echo $_SERVER["HTTP_REFERER"].'<br />'; // www.google.com
```
<Br />
### 得出结论
`$_SERVER["HTTP_X_FORWARDED_FOR"]` 获得的值是HTTP中 `X-Forwarded-For`
`$_SERVER["HTTP_CLIENT_IP"]` 获得的值是HTTP中 `CLINET-IP`( client-ip书写测试结果：服务器为window环境，则client-ip可以**大小写混写**。服务器环境为linux，则client-ip必须**全部大写**)
`$_SERVER["REMOTE_ADDR"]` 获得的值为 `最后一个跟你的服务器握手的IP`，可能会是代理IP或者其他
`$_SERVER["HTTP_REFERER"]` 获得的值为 HTTP中的`Referer`
<Br />
总的来说
`HTTP_*`容易伪造，改变HTTP请求头即可。
一些更具体的可以参照这篇问答：[获取客户端IP ，HTTP_CLIENT_IP 是一个骗局吗?](https://segmentfault.com/q/1010000000686700)
