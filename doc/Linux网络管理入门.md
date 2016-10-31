# Linux网络管理入门

本文还 **未完成**，敬请期待...

## 网络基础

### IP地址和子网

#### VLSM

VLSM（Variable Length Subnet Mask，可变长子网掩码）规定了如何在一个进行了子网划分的网络中的不同部分使用不同的子网掩码。


#### CIDR

CIDR（无类别域间路由，Classless Inter-Domain Routing）是一个在Internet上创建附加地址的方法，这些地址提供给服务提供商（ISP），再由ISP分配给客户。CIDR将路由集中起来，使一个IP地址代表主要骨干提供商服务的几千个IP地址，从而减轻Internet路由器的负担。

#### NAT


### 路由和算法、协议

#### 路由分类

#### 路由表

#### 路由算法

#### 路由协议

##### RIP路由协议

##### OSPF路由协议

##### IS-IS路由协议

##### BGP

### 其它

#### ARP协议及ARP表



##### arping

##### arp-scan

#### ICMP协议

#### DNS服务

#### DHCP服务

#### 邮件协议 SMTP、POP3、IMAP4

#### VPN服务

#### 链路层协议PPP

## Linux网络配置

### 常用网络管理配置、命令

#### IP地址管理、内核网络接口管理

#### 路由及内核路由表管理

#### ICMP和DNS工具

#### 相关配置文件

##### /etc/hosts

通过`/etc/hosts`重写主机名查找。通过该文件配置主机IP和主机名的对应关系。

    127.0.0.1 search.aicode.cc

添加如上配置之后，主机访问`search.aicode.cc`的时候，dns查询将会直接返回127.0.0.1的ip地址。该配置参数在开发的时候会经常使用，用于为开发的服务指定一个统一的测试域名。


##### /etc/resolv.conf

在该文件中指定DNS服务器，它包含了主机的域名搜索顺序和dns服务器的地址，该文件看起来如下

    search mydomain.example.com example.com
    nameserver 10.0.2.3
    nameserver 8.8.8.8
    
其中`nameserver`指定了dns服务器的IP地址，可以指定多个，查询的时候会依据提供的顺序查找，直到查找到才结束。`search`指定了残缺主机名（仅有主机名的第一部分，如myserver）的查找方式。


##### /etc/nsswitch.conf

该文件掌管了一些域名相关的优先级设定，

    hosts:    files dns
    
files在dns前，说明先查找`/etc/hosts`，然后再查找DNS服务器。

#### 常用网络调试命令

##### ping

##### traceroute & tracepath

##### mtr

mtr命令把ping命令和tracepath命令合成了一个。mtr会持续发包，并显示每一跳ping所用的时间。

    sudo mtr aicode.cc

显示效果

![mtr-aicode](media/14683732446269/mtr-aicode.gif)


##### host

host命令用来做DNS查询，如果命令参数是域名，命令会输出关联的IP，如果命令参数是IP，命令则输出关联的域名。

![host](media/14683732446269/host.jpg)


##### whois

输出指定站点的whois信息，比如站点的注册信息以及持有人等信息。

![whois](media/14683732446269/whois.gif)


##### ifconfig & ifdown & ifup

##### netstat

该命令用于显示网络状态，输出网络连接，路由表，接口统计，无效连接以及多播成员信息。

默认情况下，`netstat`命令输出出所有打开的socket信息，如果不指定任何地址族的话，则将会输出所有配置的地址族中活动的socket。

> 注意，该命令已经过时了，使用`ss`命令代替`netstat`，`ip route`取代`netstat -r`，`ip -s link`取代`netstat -i`，`ip maddr`取代`netstat -g`。

常用参数：

- **-r**，**--route** 显示内核路由表，`netstat -r`和`route -e`输出相同的内容
- **-g**，**--groups** 显示IPV4和IPV6的多播组成员信息。
- **-i**，**--interface=iface**，**-I=face** 显示所有网络接口或者指定的接口的路由表
- **-M**，**--masquerade** 显示无效连接
- **-s**，**--statistics** 显示每一个协议的统计信息
- **-n**，**--numeric** 显示IP地址取代域名、端口
- **-e**，**--extend** 显示更加详细的信息，要显示最多的信息，使用该选项两次
- **-p**，**--program** 显示每个socket所属的进程的PID和名称
- **-l**，**--listening** 只显示处于监听状态的socket
- **-a**，**--all** 显示监听中的和未监听的socket，同时使用`--interfaces`选项的话，会显示没有激活的接口
- **-C** 输出路由缓存中的路由信息
- **-t**，**-u**，**-x** 只显示TCP、UDP、UNIX套接字信息

下图显示了执行`netstat`不加任何参数的输出

![](media/14683732446269/14687429517897.jpg)

执行`netstat`命令后，会输出多列信息：

- **Proto** 显示了socket使用的协议（tcp, udp, udpl, raw）
- **Recv-Q** 接收队列，`Established`状态时指的用户进程尚未拷贝到socket的字节数，`Listening`状态时为当前积压的syn
- **Send-Q** 发送队列，`Established`状态时指远端尚未接收的字节数，`Listening`状态时为积压syn的最大值
- **Local Address** 本地socket的端口和地址，默认显示主机名和端口对应的服务名，指定`-n`选项时显示为ip+端口
- **Foreign Address** 远端socket的地址和端口，与**Local Address**相对
- **State** socket的状态，如果是原始套接字和UDP，该列为空
- **User** socket属主的用户名或者UID
- **PID/Program name** socket所属的进程id和进程名称，查看不属于当前用户的进程需要使用超级用户权限

比如执行命令`netstat -antp`，将会输出所有监听、未监听的使用tcp(t)协议的socket(a)，本地地址和远端地址使用ip形式(n)，同时列出socket所属的进程(p)

![](media/14683732446269/14687719993172.jpg)

##### route

显示、维护IP路由表。`route`命令用于维护内核IP路由表，主要用来建立某个网络接口到指定主机或者网络的静态路由。当使用`add`和`del`选项的时候可以用来维护路由表，默认是显示当前路由表的内容。

![route-n](media/14683732446269/route-n.jpg)

> 注意，在Mac上使用`netstat -r`查看路由规则。

在Mac下添加路由规则

    sudo route add 10.0.0.0/8 10.5.85.1

前面的10.0.0.0/8为网络地址，后面的10.5.85.1为网关。

##### ip

##### dig

##### nslookup

`nslookup`命令用于查询DNS，它有两种模式：交互模式和非交互模式，在交互模式下，用户可以查询多个域名或者主机的DNS信息，非交互模式则只输出指定域名或主机的信息。

下图是使用`nslookup aicode.cc`查看域名`aicode.cc`的DNS信息。

![](media/14683732446269/14688494752732.jpg)


##### ss




### 防火墙

防火墙是一种软件和（或）硬件的配置，处于互联网和小型网络之间的路由器之中，以确保小型网络免受来自互联网的攻击。你也可以为每一台机器都配置防火墙，这样每台机器就可以从数据包级别筛选进出的数据，为每台服务器配置防火墙，这叫做IP包过滤。

在Linux中，防火墙的规则是链型的。一个表就是一套链，数据包在Linux网络子系统的各部分移动时，内核就会对包应用某套规则。这些数据结构由内核来维护，这个系统叫做 **iptables**，可以使用用户控件的命令`iptables`来创建和修改规则。

#### iptables入门

Linux防火墙由netfilter和iptables两部分组成，netfilter组件是内核的一部分，由一些信息包过滤表组成，包含了内核用于控制信息包过滤处理的规则。iptables则是一种工具，它工作在用户空间，使用它可以对内核的包过滤规则进行控制。

> 由于NAT也是通过包过滤规则进行配置，因此iptables也可以用来配置NAT





iptables共设置了五个规则链，分别处于内核空间的五个位置：

- **PREROUTING** 路由前
- **INPUT** 数据包从内核空间流入用户空间，数据包入口
- **FORWARD** 将数据包转发到本机的其它网卡设备
- **OUTPUT** 数据包出口
- **POSTROUTING** 路由后


#### firewall


## 常见网络协议

### NTP



---

参考：

- [精通Linux（第二版）](https://book.douban.com/subject/26546893/) 
- [百度百科-无类域间路由](http://baike.baidu.com/view/4217886.htm?fromtitle=CIDR&fromid=3695195&type=search)
- [百度百科-可变长子网掩码](http://baike.baidu.com/view/1252848.htm)


