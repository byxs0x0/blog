---
title: CTF-WEB题目链接
date: 2018-03-07 10:28:07
tags:
  - vul
  - ctf
---
里面合集了CTF平台中WEB的题目，并且做了简单的分类，为了方便后期教学，能更方便的找到题目去训练。  
**鼠标移到链接上，可以查看tips**

# 平台
[CG-CTF](https://cgctf.nuptsast.com/challenges#Web) -- [南京邮电大学网络攻防训练平台](http://ctf.nuptzj.cn/challenges)
[BugKu](http://ctf.bugku.com/challenges)


# 基本
**CG-CTF**
[签到题](http://chinalover.sinaapp.com/web1/)
[这题不是WEB](http://chinalover.sinaapp.com/web2/index.html "真的，你要相信我！这题不是WEB")
[层层递进](http://chinalover.sinaapp.com/web3/)
[AAencode](http://chinalover.sinaapp.com/web20/aaencode.txt  "javascript aaencode")
[伪装者](http://chinalover.sinaapp.com/web4/xxx.php)
[Header](http://way.nuptzj.cn/web5/  "头啊！！头啊！！！")

**BugKu-CTF**
[Web2](http://120.24.86.145:8002/web2/)
[web基础GET](http://120.24.86.145:8002/get/)
[web基础POST](http://120.24.86.145:8002/post/)
[域名解析](http://flag.bugku.com "听说把 flag.bugku.com 解析到120.24.86.145 就能拿到flag")
[你必须让他停下](http://120.24.86.145:8002/web12/)
[web5](http://120.24.86.145:8002/web5/)
[过狗一句话]()

# 爆破
**BugKu-CTF**
[输入密码查看flag](http://120.24.86.145:8002/baopo/)
[细心](http://120.24.86.145:8002/web13/ "想办法变成admin")

# python
**BugKu-CTF**
[秋名山老司机](http://120.24.86.145:8002/qiumingshan/)
[速度要快](http://120.24.86.145:8002/web6/)
[cookies欺骗](http://120.24.86.145:8002/web11/)
[]()

# SQLi
**CG-CTF**
[MYSQL](http://chinalover.sinaapp.com/web11/ "不能每一题都这么简单嘛
你说是不是？")
[GBK Injection]()
[SQL注入1](http://chinalover.sinaapp.com/index.php  "听说你也会注入？")
[SQL Injection](http://chinalover.sinaapp.com/web15/index.php "反斜杠可以用来转义
仔细查看相关函数的用法")
[SQL注入2](http://4.chinalover.sinaapp.com/web6/index.php  "注入第二题~~主要考察union查询")

**BugKu-CTF**
[sql注入](http://103.238.227.13:10083/)
[成绩单](http://120.24.86.145:8002/chengjidan/)
[login1(SKCTF)](http://118.89.219.210:49163/ "hint:SQL约束攻击")
[INSERT INTO注入](http://120.24.86.145:8002/web15/ "提示给了源码，到题目看比较好")

# XSS
**BugKu-CTF**
[XSS](http://103.238.227.13:10089/)
[]()
[]()
[]()

# 前端
**CG-CTF**
[签到2](http://teamxlc.sinaapp.com/web1/02298884f0724c04293b4d8c0178615e/index.php)
[单身二十年](http://chinalover.sinaapp.com/web8/  "这题可以靠技术也可以靠手速！
老夫单身二十年，自然靠的是手速！")
[单身一百年也没用](http://chinalover.sinaapp.com/web9/  "是的。。这一题你单身一百年也没用")

**BugKu-CTF**
[计算器](http://120.24.86.145:8002/yanzhengma/)
[web3](http://120.24.86.145:8002/web3/)
[web4](http://120.24.86.145:8002/web4/  "看看源代码吧")
[点击一百万次](http://120.24.86.145:9001/test/ "hints:JavaScript")

# 文件包含
**CG-CTF**
[文件包含](http://4.chinalover.sinaapp.com/web7/index.php  "没错 这就是传说中的LFI")

**BugKu-CTF**
[本地包含](http://120.24.86.145:8003/)
[flag在index里](http://120.24.86.145:8005/post/)
[文件包含2](http://118.89.219.210:49166/ "hint:文件包含")
[]()

# 文件上传
**CG-CTF**
[上传绕过](http://teamxlc.sinaapp.com/web5/21232f297a57a5a743894a0e4a801fc3/index.html  "猜猜代码怎么写的")

**BugKu-CTF**
[文件上传测试](http://103.238.227.13:10085/)
[求getshell](http://120.24.86.145:8002/web9/)
[]()
[]()

# 任意文件下载
**CG-CTF**
[Download~!](http://way.nuptzj.cn/web6/  "想下啥就下啥~别下音乐，不骗你，试试下载其他东西~")

# 越权
**CG-CTF**
[COOKIE](http://chinalover.sinaapp.com/web10/index.php  "0==not")
[密码重置](http://nctf.nuptzj.cn/web13/index.php?user1=Y3RmdXNlcg%3D%3D  "重置管理员账号：admin 的密码
你在点击忘记密码之后 你的邮箱收到了这么一封重置密码的邮件：")

**BugKu-CTF**
[]()
[]()
[]()
[]()

# 代码审计
**CG-CTF**
[md5 collision](http://chinalover.sinaapp.com/web19/)
[php decode]() 提示是一串代码，去平台查看
[/x00](http://teamxlc.sinaapp.com/web4/f5a14f5e6e3453b78cd73899bad98d53/index.php "题目有多种解法，你能想出来几种？")
[bypass again](http://chinalover.sinaapp.com/web17/index.php  "依旧是弱类型")
[变量覆盖](http://chinalover.sinaapp.com/web18/index.php)
[PHP是世界上最好的语言](http://way.nuptzj.cn/php/index.php)
[pass check]() 提示是一串代码，去平台查看
[起名字真难]() 提示是一串代码，去平台查看
[变量覆盖]()

**BugKu-CTF**
[矛盾](http://120.24.86.145:8002/get/index1.php)
[变量1](http://120.24.86.145:8004/index1.php)
[备份是个好习惯](http://120.24.86.145:8002/web16/ "听说备份是个好习惯")
[never give up](http://120.24.86.145:8006/test/hello.php)
[字符？正则？](http://120.24.86.145:8002/web10/)
[前女友(SKCTF)](http://118.89.219.210:49162/)
[各种绕过](http://120.24.86.145:8002/web8/)
[WEB8](http://120.24.86.145:8002/web8/)

# 其他姿势
**CG-CTF**
[file_get_contents]()

**BugKu-CTF**
[]()
[]()
[]()
[]()

# 综合
**CG-CTF**
[综合题](http://teamxlc.sinaapp.com/web3/b0b0ad119f425408fc3d45253137d33d/index.php  "tip:bash")
[综合题2](http://cms.nuptzj.cn/ "非xss题 但是欢迎留言~")
[密码重置2](http://nctf.nuptzj.cn/web14/index.php "TIPS:
1.管理员邮箱观察一下就可以找到
2.linux下一般使用vi编辑器，并且异常退出会留下备份文件
3.弱类型bypass")
**BugKu-CTF**
[welcome to bugkuctf](http://120.24.86.145:8006/test1/)
[]()
[]()
[]()
