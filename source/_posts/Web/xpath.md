---
title: XPath注入小结
date: 2018-06-06 16:33:45
categories: Web
tags:
description:
---

>在很多WEB应用中常常会用XML格式来储存数据（国外占多数），而XPATH就是一个XML文档中解析和提取数据的查询语言（类似于SQL语言）。如果我们能控制一个XAPTH输入点，那么我们可以像注入SQL一样的去注入XPATH，当然你不能用SQL语句以及UNION联合查询。
<!-- more -->

# XPath知识储备
XPath就好比SQL查询语句，能够对DOM树进行查询操作，并获取相应结果。


# XPath注入原理
XPath原理和SQL注入大体相似，利用XPath语法构建注入代码，对存储数据的XML文档进行注入，从而获取正常途径无法获取的到数据内容。

比如一个网站某应用程序将数据保存在 XML 中，并且没有对用户的输入做限制和处理。攻击者构建注入语句就插入到 XPath 查询中，即产生 XPath 注入，那么攻击者就可以通过控制查询来获取数据，或者删除数据之类的操作


```
正常查询语句:
//users/user[loginID/text()='ccc'and password/text()='test123']

如果loginId字段和password字段都输入为'or 1=1 or ''='

那么语句变为：
//users/user[loginID/text()=''or 1=1 or ''='' and password/text()='' or 1=1 or ''='']

条件永真，便可以获取所有数据

// 某站点保存帐号密码的XML数据内容
<?xml version="1.0" encoding="UTF-8"?>
<users>
  <user>
    <firstname>Ben</firstname>
    <lastname>Elmore</lastname>
    <loginID>ccc</loginID>
    <password>test123</password>
  </user>
  <user>
    <firstname>Shlomy</firstname>
    <lastname>Gantz</lastname>
    <loginID>xyz</loginID>
    <password>123test</password>
  </user>
  ...
  ...
</users>
```

# XPath利用


# 参考
[XXE & XPath #101](https://github.com/PyxYuYu/MyBlog/issues/101)
