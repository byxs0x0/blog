---
title: Python Hacker
date: 2018-05-05 16:21:41
categories: Python
tags: Python
description:
---

玩一下python hacker
<!-- more -->
# 下载与安装
环境：Ubuntu16.04
官网下载最新版[scapy](https://pypi.org/project/scapy/#files)
```bash
cd scapy-2.4.0
sudo python setup.py install # python2
sudo python3 setup.py install # python3
```

<br>

# 启动scapy
```bash
sudo scapy # 需要root权限才能发送数据包
```

<br>

# 基本使用

## 构建数据包
根据OSI参考模型，利用`/`来组合为一个数据包。
```
>>> Ether()/IP()/ICMP()
<Ether  type=0x800 |<IP  frag=0 proto=icmp |<ICMP  |>>>
>>> IP()/TCP()/"GET / HTTP/1.1\r\n\r\n"
<IP  frag=0 proto=tcp |<TCP  |<Raw  load='GET / HTTP/1.1\r\n\r\n' |>>>
```

### 构建一组数据包
```
In [5]: a = IP(dst="byxs0x0.cn/28")

In [6]: a
Out[6]: <IP  dst=Net('byxs0x0.cn/28') |>

In [7]: [p for p in a]
Out[7]:
[<IP  dst=104.194.80.96 |>,
 <IP  dst=104.194.80.97 |>,
 <IP  dst=104.194.80.98 |>,
 <IP  dst=104.194.80.99 |>,
 <IP  dst=104.194.80.100 |>,
 <IP  dst=104.194.80.101 |>,
 <IP  dst=104.194.80.102 |>,
 <IP  dst=104.194.80.103 |>,
 <IP  dst=104.194.80.104 |>,
 <IP  dst=104.194.80.105 |>,
 <IP  dst=104.194.80.106 |>,
 <IP  dst=104.194.80.107 |>,
 <IP  dst=104.194.80.108 |>,
 <IP  dst=104.194.80.109 |>,
 <IP  dst=104.194.80.110 |>,
 <IP  dst=104.194.80.111 |>]


In [15]: b = IP(ttl=[1, 2, (5, 7)])
In [16]: [p for p in b]
Out[16]:
[<IP  ttl=1 |>,
 <IP  ttl=2 |>,
 <IP  ttl=5 |>,
 <IP  ttl=6 |>,
 <IP  ttl=7 |>]
```

### 发送ARP包
```
>>> ls(ARP)
hwtype     : XShortField                         = (1)         1表示以太网
ptype      : XShortEnumField                     = (2048)      上层协议，一般为0x0800(IP)
hwlen      : ByteField                           = (6)         MAC地址长度
plen       : ByteField                           = (4)         IP地址长度
op         : ShortEnumField                      = (1)         ARP数据包类型，1为请求，2为应答
hwsrc      : ARPSourceMACField                   = (None)      发送端MAC地址
psrc       : SourceIPField                       = (None)      发送端IP地址
hwdst      : MACField                            = ('00:00:00:00:00:00')  目标段MAC地址
pdst       : IPField                             = ('0.0.0.0') 目标段IP地址

eth = Ether(src="bc:77:37:d7:cd:14",type=0x0806)
arp = ARP(hwtype=0x1,ptype=0x0800,op=0x1,hwsrc="bc:77:37:d7:cd:14",\
          psrc="192.168.1.107",pdst="192.168.1.108")
package = eth/arp
recv = srp1(package)
recv.show()
```



### 发送ICMP包
```
ans,unans = sr(IP(dst="192.168.1.108")/ICMP())
ans.show()
```

### 发送TCP包
```
recv = sr1(IP(dst="220.181.51.53")/TCP(dport=80,flags="S"))
recv.show()

如果端口开放则返回flags为SA（SYN/ACK）
如果没开放则返回flags为RA（RST/ACK）
```


[具体看着](http://burningcodes.net/scapy%E6%94%B6%E5%8F%91%E6%95%B0%E6%8D%AE%E5%8C%85%E6%80%BB%E7%BB%93/)

### 常用函数
RandMAC()  任意MAC地址
RandIP()   任意IP地址
RandInt()  `IP(id=RandInt())`
fuzz()   更改一些默认的不可以被计算的值（比如校验和checksums）,更改的值是随机的，但是类型是符合字段的值的。例如`send(IP(dst=" www.baidu.com")/fuzz(UDP()/NTP(version=4)),loop=2)`


## 发送与接收数据包
 - `send()` 在第3层发送数据包。也就是说它会为你处理路由和第2层的数据。
 - `sendp()` 工作在第2层。选择合适的接口和正确的链路层协议都取决于你。
 - `sr()` 该函数返回有回应的数据包和没有回应的数据包,一个是answer list 另一个是unanswered list
 - `srloop()` 在三层循环发送一个数据包，并且打印答案
 - `sr1()` 一种变体，只返回应答发送的分组（或分组集）。发送的数据包必须是第3层报文（IP，ARP等）
 - `srp()` 则是使用第2层报文（以太网，802.3等）


loop=1       循环发送
inter=1      每隔1秒发送
timeout=1    超时1秒就丢弃，实际时间看程序处理能力而定
retry=3      会对无应答的数据包重复发送三次。retry=-3，则会一直发送无应答的数据包。


## 信息查看
lsc()列出scapy支持的所有的命令。
conf 显示所有的配置信息。conf变量保存了scapy的配置信息。
help()显示某一命令的使用帮助，如help(Ether)。

ls() 可以列出scapy支持的所有协议。常用的有ARP、Ether、ICMP、IP、UDP、TCP，也支持SNMP、DHCP、STP等。
ls(Ether)  查看协议结构
ls(packet) 查看数据包结构

```
>>> ls(Ether)
dst        : DestMACField                        = (None)
src        : SourceMACField                      = (None)
type       : XShortEnumField                     = (36864)
>>> ls(Ether())
WARNING: Mac address to reach destination not found. Using broadcast.
dst        : DestMACField                        = 'ff:ff:ff:ff:ff:ff' (None)
src        : SourceMACField                      = '00:0c:29:b7:6d:cd' (None)
type       : XShortEnumField                     = 36864           (36864)
```

Ether().src 查看特定字段
Ether().show() 显示指定数据包的详细信息
Ether().display() 显示指定数据包的详细信息
```
>>> Ether().src
'00:0c:29:b7:6d:cd'
>>> Ether().show()
WARNING: Mac address to reach destination not found. Using broadcast.
###[ Ethernet ]###
  dst= ff:ff:ff:ff:ff:ff
  src= 00:0c:29:b7:6d:cd
  type= 0x9000
```


sprintf()输出某一层某个参数的取值,如果变量不存在会输出`??`
```
>>> Ether().sprintf("DST MAC:%Ether.dst%")
'DST MAC:ff:ff:ff:ff:ff:ff'
>>> Ether().sprintf("DST MAC:%Ether.dsts%")
'DST MAC:??'
```
具体格式
```
%[[fmt][r],][layer[:nb].]field%

layer:协议层的名字，如Ether、IP、Dot11、TCP等。
filed:需要显示的参数。
nb:当有两个协议层有相同的参数名时，nb用于到达你想要的协议层。
r:是一个标志。当使用r标志时，意味着显示的是参数的原始值。例如，TCP标志中使用人类可阅读的字符串’SA’表示SYN和ACK标志，而其原始值是18.
```


## 读取cat内容
```
>>> test = rdpcap("./test.cap")
```

## 嗅探包
```
count=0
store=1
timeout=N
filter="tcp and host 192.168.1.1 and port 80"
iface="eth0"

>>> pkts = sniff(count=10, iface="eth0")
>>> pkts
<Sniffed: TCP:0 UDP:0 ICMP:10 Other:0>
```



# 实例

## ARPping
ARP叫做地址解析协议。就是根据主机中的ARP表将IP地址转换为MAC地址。ARP表是存放IP地址和MAC地址的映射表。

当主机A向主机B发送一个IP数据报时，先在ARP表中查看有无主机B的IP地址，如果有，根据映射关系得到主机B MAC地址。在将MAC地址写入MAC帧。
如果没有，主机A会以广播形式发送一个ARP请求分组，主机B收到后向主机A发送一个ARP响应分组。
[详细看着](https://blog.csdn.net/tigerjibo/article/details/7351992)

ARPping 就是通过发送ARP广播包，来根据应答去判断，主机是否存活。

具体实现
```python
from scapy.all import *
from threading import *
import optparse

def send_arp(ip):
    arpPkt=Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=ip, hwdst="ff:ff:ff:ff:ff:ff")
    res = srp1(arpPkt, timeout=1, verbose=0)
    if res:
        print("IP:" + res.psrc + "\t MAC:" + res.hwsrc)

parser = optparse.OptionParser()
parser.add_option('-H', dest='tgtHost', type='string', help='for example:192.168.1')
(options, args) = parser.parse_args()
for i in range(1,254):
    ip = options.tgtHost + '.' + str(i)
    t = Thread(target=send_arp, args=(ip,))
    t.start()
```

## ICMPping
```python
from scapy.all import *
from threading import *
import optparse

def send_icmp(ip):
    icmpPkt = IP(dst=ip)/ICMP()
    res = sr1(icmpPkt, timeout=1, verbose=0)
    if res:
        print("IP:" + ip)

parse = optparse.OptionParser()
parse.add_option('-H', dest='tgtHost', type='string', help='for example:192.168.1')
(options, args) = parse.parse_args()

for i in range(1, 255):
    ip = options.tgtHost + '.' + str(i)
    t = Thread(target=send_icmp, args=(ip,))
    t.start()
```

## RIP
```
RIP组播地址224.0.0.9，使用了UDP协议的520端口。

p = IP(src='172.168.1.1', dst='172.16.1.2')/UDP(sport=520, dport=520)/RIP(cmd=2)
px = RIPEntry(addr='200.200.200.200')/RIPEntry(addr='100.100.100.100', metric=1)
send(p/px)

注意：
如果跨网段欺骗，那么源地址要同一网段
"水平分割"特性

```

## OSPF
```
OSPF基于IP，协议号为89
组播地址224.0.0.5(所有OSPF路由器)和224.0.0.6(DR DBR)

有如下几种攻击方式：
DR/DBR篡改攻击  让OPSF网络重新选举DR/DBR
LSR报文伪造攻击 不断发LSR报文
序列号+1攻击   OSPF通过序列号竞选，两个路由器发生竞争
最大序列号攻击 0X7FFFFFFF
最大年龄攻击 将链路年龄设置为最大  Max Age

>>> packet = Ether(src='00:06:28:b9:85:31',dst='01:00:5e:00:00:05')
>>> packet = packet/Dot1Q(vlan=33)
>>> packet.show()
###[ Ethernet ]###
  dst= 01:00:5e:00:00:05
  src= 00:06:28:b9:85:31
  type= 0x8100
###[ 802.1Q ]###
     prio= 0
     id= 0
     vlan= 33
     type= 0x0
>>> packet = packet/IP(src='172.17.2.2',dst='224.0.0.5')
>>> packet = packet/OSPF_Hdr(src='172.17.2.2')
>>> packet = packet/OSPF_Hello(router='172.17.2.2',backup='172.17.2.1',neighbor='172.17.2.1')

>>> packet.show()
###[ Ethernet ]###
  dst= 01:00:5e:00:00:05
  src= 00:06:28:b9:85:31
  type= 0x8100
###[ 802.1Q ]###
     prio= 0
     id= 0
     vlan= 33
     type= 0x800
###[ IP ]###
        version= 4
        ihl= 0
        tos= 0x0
        len= 0
        id= 1
        flags=
        frag= 0
        ttl= 64
        proto= ospf
        chksum= 0x0
        src= 172.17.2.2
        dst= 224.0.0.5
        options= ''
###[ OSPF Header ]###
           version= 2
           type= Hello
           len= 0
           src= 172.17.2.2
           area= 0.0.0.0
           chksum= 0x0
           authtype= Null
           authdata= 0x0
           reserved= 0x0
           keyid= 1
           authdatalen= 0
           seq= 0x0
###[ OSPF Hello ]###
              mask= 255.255.255.0
              hellointerval= 10
              options=
              prio= 1
              deadinterval= 40
              router= 172.17.2.2
              backup= 172.17.2.1
              neighbor= 172.17.2.1

sendp(packet,iface='dlink')

```
