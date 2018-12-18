# tcpdump简明教程

![DSC07466](https://ssl.aicode.cc/DSC07466.jpg)

本文翻译自 [A tcpdump Tutorial and Primer with Examples][original] 一文，在使用Linux系统进行网络抓包分析的时候，一直没有找到比较简便的非图形界面的方法，在Linux系统下`tcpdump`命令确实是一柄利器，但是一直苦于学习成本较高，迟迟没有下手。看了 [A tcpdump Tutorial and Primer with Examples][original] 这篇文章之后，发现其实使用`tcpdump`也没有那么困难，特别是其导出的cap文件，再使用wireshark等图形界面软件打开分析非常方便。因此，将其翻译出来，一方面方便自己学习，一方面也为像我一样对tcpdump感兴趣的人提供一个学习途径。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

[TOC]

## 概述

对于专业的信息安全人员来说，**tcpdump** 是非常重要的网络分析工具。对于任何想深入理解TCP/IP的人来说，掌握该工具的使用时非常必要的。很多人更喜欢高级的分析工具，比如Wireshark，但我相信通常情况下这是个错误的选择。

当使用工具对网络进行分析的时候，更重要的是人对结果的分析，而不是应用的分析。这就促使了对TCP/IP协议栈的理解，因此，我强烈建议学会使用 **tcpdump** 代替其它工具。

    15:31:34.079416 IP (tos 0x0, ttl 64, id 20244, offset 0, flags [DF], 
    proto: TCP (6), length: 60) source.35970 > dest.80: S, cksum 0x0ac1 
    (correct), 2647022145:2647022145(0) win 5840 0x0000: 4500 003c 4f14 
    4006 7417 0afb 0257  E..  0x0010: 4815 222a 8c82 0050 9dc6 5a41 0000 
    0000  H."*...P..ZA.... 0x0020: a002 16d0 0ac1 0000 0204 05b4 
    0402 080a  ................ 0x0030: 14b4 1555 0000 0000 0103 0302

TABLE 1. 原生 TCP/IP 输出

## 基础

下面是一些用来配置 **tcpdump** 的选项，它们非常容易被遗忘，也容易和其它类型的过滤器比如Wireshark等混淆。

### 选项

- **-i any** 监听所有的网卡接口，用来查看是否有网络流量
- **-i eth0** 只监听eth0网卡接口
- **-D** 显示可用的接口列表
- **-n** 不要解析主机名
- **-nn** 不要解析主机名或者端口名
- **-q** 显示更少的输出(更加quiet)
- **-t** 输出可读的时间戳
- **-tttt** 输出最大程度可读的时间戳
- **-X** 以hex和ASCII两种形式显示包的内容
- **-XX** 与**-X**类似，增加以太网header的显示
- **-v, -vv, -vvv** 显示更加多的包信息
- **-c** 只读取x个包，然后停止
- **-s** 指定每一个包捕获的长度，单位是byte，使用`-s0`可以捕获整个包的内容
- **-S** 输出绝对的序列号
- **-e** 获取以太网header
- **-E** 使用提供的秘钥解密IPSEC流量

### 表达式

在**tcpdump**中，可以使用表达式过滤指定类型的流量。有三种主要的表达式类型：**type**，**dir**，**proto**。

- 类型（type）选项包含：**host**，**net**，**port**
- 方向（dir）选项包含：**src**，**dst**
- 协议（proto）选项包含：**tcp**，**udp**，**icmp**，**ah**等

## 示例

### 捕获所有流量

查看所有网卡接口上发生了什么

    tcpdump -i any

### 指定网卡接口

查看指定网卡上发生了什么

    tcpdump -i eth0

### 原生输出

查看更多的信息，不解析主机名和端口号，显示绝对序列号，可读的时间戳

    tcpdump -ttttnnvvS
    
### 查看指定IP的流量

这是最常见的方式，这里只查看来自或者发送到IP地址1.2.3.4的流量。

    tcpdump host 1.2.3.4

### 查看更多的包信息，输出HEX

当你需要查看包中的内容时，使用hex格式输出是非常有用的。

    # tcpdump -nnvXSs 0 -c1 icmp

    tcpdump: data link type PKTAP
    tcpdump: listening on pktap, link-type PKTAP (Apple DLT_PKTAP), capture size 262144 bytes
    16:08:16.791604 IP (tos 0x0, ttl 64, id 34318, offset 0, flags [none], proto ICMP (1), length 56)
        192.168.102.35 > 114.114.114.114: ICMP 192.168.102.35 udp port 50694 unreachable, length 36
    	IP (tos 0x0, ttl 152, id 0, offset 0, flags [none], proto UDP (17), length 112)
        114.114.114.114.53 > 192.168.102.35.50694: [|domain]
    	0x0000:  5869 6c88 7f64 784f 4392 ed7e 0800 4500  Xil..dxOC..~..E.
    	0x0010:  0038 860e 0000 4001 e906 c0a8 6623 7272  .8....@.....f#rr
    	0x0020:  7272 0303 3665 0000 0000 4500 0070 0000  rr..6e....E..p..
    	0x0030:  0000 9811 16cd 7272 7272 c0a8 6623 0035  ......rrrr..f#.5
    	0x0040:  c606 005c 0000                           ...\..
    1 packet captured
    357 packets received by filter
    0 packets dropped by kernel

### 使用源和目的地址过滤

    tcpdump src 2.3.4.6
    tcpdump dst 3.4.5.6

### 过滤某个子网的数据包

    tcpdump net 1.2.3.0/24
    
### 过滤指定端口相关的流量

    tcpdump port 3389
    tcpdump src port 1025

### 过滤指定协议的流量

    tcpdump icmp

### 只显示IPV6流量

    tcpdump ip6

### 使用端口范围过滤

    tcpdump portrange 21-23

### 基于包的大小过滤流量

    tcpdump less 32
    tcpdump greater 64
    tcpdump <=128

### 将捕获的内容写入文件

使用`-w`选项可以将捕获的数据包信息写入文件以供以后分析，这些文件就是著名的PCAP(PEE-cap)文件，很多应用都可以处理它。

    tcpdump port 80 -w capture_file

使用tcpdump加载之前保存的文件进行分析

    tcpdump -r capture_file

## 高级

使用组合语句可以完成更多高级的过滤。

- AND: **and** or **&&**
- OR: **or** or **||**
- EXCEPT: **not** or **!**

### 过滤指定源IP和目的端口

    tcpdump -nnvvS src 10.5.2.3 and dst port 3389

### 过滤指定网络到另一个网络

比如下面这个，查看来自192.168.x.x的，并且目的为10.x或者172.16.x.x的所有流量，这里使用了hex输出，同时不解析主机名

    tcpdump -nvX src net 192.168.0.0/16 and dst net 10.0.0.0/8 or 172.16.0.0/16

### 过滤到指定IP的非ICMP报文

    tcpdump dst 192.168.0.2 and src net and not icmp

### 过滤来自非指定端口的指定主机的流量

下面这个过滤出所有来自某个主机的非ssh流量

    tcpdump -vv src mars and not dst port 22

### 复杂分组和特殊字符

当构建复杂的过滤规则的时候，使用单引号将规则放到一起是个很好的选择。特别是在包含**()**的规则中。比如下面的规则就是错误的，因为括号在shell中会被错误的解析，可以对括号使用`\`进行转义或者使用单引号

    tcpdump src 10.0.2.3 and (dst port 3389 or 22)
    
应该修改为

    tcpdump 'src 10.0.2.3 and (dst port 3389 or 22)'

### 隔离指定的TCP标识

可以基于指定的TCP标识（flag）来过滤流量。

> 下面的过滤规则中，**tcp[13]**表示在TCP header中的偏移位置13开始，后面的数字代表了匹配的byte数。

#### 显示所有的URGENT (URG)包

    tcpdump 'tcp[13] & 32!=0'

#### 显示所有的ACKNOWLEDGE (ACK)包

    tcpdump 'tcp[13] & 16!=0'

#### 显示所有的PUSH(PSH)包

    tcpdump 'tcp[13] & 8!=0'

#### 显示所有的RESET(RST)包

    tcpdump 'tcp[13] & 4!=0'

#### 显示所有的SYNCHRONIZE (SYN) 包

    tcpdump 'tcp[13] & 2!=0'

#### 显示所有的FINISH(FIN)包

    tcpdump 'tcp[13] & 1!=0'

#### 显示说有的SYNCHRONIZE/ACKNOWLEDGE (SYNACK)包

    tcpdump 'tcp[13]=18'

#### 其它方式
    
与大多数工具一样，也可以使用下面这种方式来捕获指定TCP标识的流量

    tcpdump 'tcp[tcpflags] == tcp-syn'
    tcpdump 'tcp[tcpflags] == tcp-rst'
    tcpdump 'tcp[tcpflags] == tcp-fin'

### 识别重要流量

最后，这里有一些重要的代码片段你可能需要，它们用于过滤指定的流量，例如畸形的或者恶意的流量。

#### 过滤同时设置SYN和RST标识的包（这在正常情况下不应该发生）

    tcpdump 'tcp[13] = 6'

#### 过滤明文的HTTP GET请求

    tcpdump 'tcp[32:4] = 0x47455420'

#### 通过横幅文本过滤任意端口的SSH连接

    tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482D'

#### 过滤TTL小于10的包（通常情况下是存在问题或者在使用traceroute）

    tcpdump 'ip[8] < 10'

#### 过滤恶意的包

    tcpdump 'ip[6] & 128 != 0'

## 补充（非原文内容）

下面这个命令用于过滤所有与8080端口相关的tcp流量，将其输出到capcha.cap文件中，我们可以使用wireshark打开这个文件，更加可视化的分析过滤其中包含的http流量。

    tcpdump -tttt -s0 -X -vv tcp port 8080 -w captcha.cap

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 原文

- [A tcpdump Tutorial and Primer with Examples][original]


[original]: https://danielmiessler.com/study/tcpdump/

