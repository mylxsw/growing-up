# Keepalived & LVS 搭建高可用的Web服务

![](https://oayrssjpa.qnssl.com/15362160228382.jpg)


[TOC]

在本文中，我将会讲述如何在Centos 7下基于Keepalived和LVS技术，实现Web服务的高可用和负载均衡，我们的目标拓扑结构如下图所示

![未命名文件](https://oayrssjpa.qnssl.com/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.jpg)

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 准备

> 如果你觉得一步一步按照下面的操作来搭建太过麻烦，可以直接下载 [mylxsw/keepalived-example](https://github.com/mylxsw/keepalived-example) 项目，然后执行 `make create` 即可一键搭建起整个演示环境。

使用Vagrant创建四台虚拟机用于测试使用，[Vagrant][vagrant] 配置文件格式如下

    Vagrant.configure("2") do |config|
      config.vm.box = "centos/7"
      config.vm.network "private_network", ip: "IP地址"
    end

对于每个配置，需要替换配置文件中的IP地址

| 目录 | IP | 用途 |
| --- | --- | --- |
| keepalived | 192.168.88.8 | 负载均衡Master |
| keepalived-backup | 192.168.88.9 | 负载均衡Backup |
| node-1 | 192.168.88.10 | web服务器 |
| node-2 | 192.168.88.11 | web服务器 |
| client | 192.168.88.2 | 客户端，也可以直接用自己的电脑，IP地址任意都可 |

VIP为 `192.168.88.100`，客户端IP为 `192.168.88.2`。

> 启动Vagrant服务器需要进入服务器所在目录，执行 `vagrant up` 命令，登录到服务器需要执行 `vagrant ssh` 命令。如果你还没有接触过Vagrant，那么可以看看这篇文章 [Vagrant入门][vagrant-tutorial]。由于本文中很多命令都需要使用 `root` 权限进行操作，因此建议执行命令 `su root` 直接提升到root权限（密码为 **vagrant** ），否则需要在所有命令前添加 `sudo` 来执行。

分别登录每台服务器，设置其`hostname`，方便后面我们区分不同的服务器

    # 在192.168.88.8上执行
    hostnamectl set-hostname keepalived
    # 在192.168.88.9上执行
    hostnamectl set-hostname keepalived-backup
    # 在192.168.88.10上执行
    hostnamectl set-hostname node-1
    # 在192.168.88.11上执行
    hostnamectl set-hostname node-2
    
然后退出重新登录，就可以看到hostname生效了。

### 安装Keepalived服务

在 **keepalived** 和 **keepalived-backup** 两个虚拟机上，安装keepalived服务

    yum install -y keepalived ipvsadm

安装完成后，可以看到生成了`/etc/keepalived/keepalived.conf`配置文件，不过这个文件是Keepalived提供的示例，后面我们需要修改。

将 Keepalived 服务添加到开机自启动

    systemctl enable keepalived

### 安装web服务

在两台web服务器 **node-1** 和 **node-2** 上，我们就简单安装一个 **nginx**，然后开放80端口，提供简单的web服务用于测试

    yum install yum-utils
    yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
    yum install -y openresty

为了方便查看效果，我们将 nginx 的默认web页面修改为显示服务器的IP

    ip addr show eth1 | grep '192.168.88.' | awk '{print $2}' > /usr/local/openresty/nginx/html/index.html

然后，启动web服务

    systemctl enable openresty
    systemctl start openresty

## Keepalive实现高可用

在这里，我们实现Keepalive服务（**keepalived** 和 **keepalived-backup** 两台服务器）的高可用，也就是为负载均衡服务提供主备服务，当master挂掉之后，backup自动成为新的master继续提供服务。

下面是Keepalived配置文件内容

    global_defs {
        router_id LVS_8808
    }
    
    vrrp_instance HA_WebServer {
        state MASTER
        ! 监听的网卡，keepalived会将VIP绑定到该网卡上
        interface eth1
        ! 虚拟路由器ID，同一个实例保持一致，多个实例不能相同
        virtual_router_id 18
        garp_master_refresh 10
        garp_master_refresh_repeat 2
        ! VRRP 优先顺序的设定值。在选择主节点的时候，该值大的备用 节点会优先漂移为主节点
        priority 100
        ! 发送VRRP通告的间隔。单位是秒
        advert_int 1
    
        ! 集群授权密码
        authentication {
            auth_type PASS
            auth_pass 123456
        }
    
        ! 这里是我们配置的VIP，可以配置多个，每个IP一行
        virtual_ipaddress {
             192.168.88.100/24
        }
    }

两台 keepalived服务器均使用该配置文件，唯一的不同是 `priority` 的取值在两台服务器上是不同的，我们设置 **keepalived** 服务器的 `priority=100`，**keepalived-backup** 的 `priority=99`。

> 两台服务器设置不同的优先级之后，只要两台服务器都没有正常工作，则优先级高的为 主服务器，优先级低的为 备服务器。

配置完成后，重启 keepalived 服务

    systemctl restart keepalived

然后，我们可以看到 **keepalived** 服务器绑定了VIP **192.168.88.100**

![](https://oayrssjpa.qnssl.com/15362090253889.jpg)

**keepalived** 服务器

![-w614](https://oayrssjpa.qnssl.com/15362042379187.jpg)
    
**keepalived-backup**服务器
 
 ![-w615](https://oayrssjpa.qnssl.com/15362042587330.jpg)          
 
我们验证一下主服务器挂掉之后，备份服务器是否能够正常接替工作，在 **keepalived** 服务器上，执行 `systemctl stop keepalived` 命令，停止keepalived服务，模拟服务器挂掉的情景，然后我们看到

![](https://oayrssjpa.qnssl.com/15362090457018.jpg)


**keepalived** 服务器

![-w616](https://oayrssjpa.qnssl.com/15362042064293.jpg)

**keepalived-backup**服务器
           
![-w632](https://oayrssjpa.qnssl.com/15362041802216.jpg)


VIP成功漂移到了备份服务器，在备份服务器的`/var/log/message`日志中，可以看到如下信息

![-w858](https://oayrssjpa.qnssl.com/15362041371710.jpg)

重启 **keepalived** 服务器的Keepalived服务（`systemctl start keepalived`），模拟服务器恢复，我们可以看到VIP又重新漂移回了 **keepalived** 服务器（因为 **keepalived** 服务器设置的 `priority` 大于 **keepalived-backup** 服务器的设置）。查看 **keepalived-backup** 的日志，可以看到下面信息

![-w1033](https://oayrssjpa.qnssl.com/15362044690887.jpg)


## LVS实现负载均衡

在两台Keepalived服务器的配置文件 `/etc/keepalived/keepalived.conf` 中，追加以下配置

    virtual_server 192.168.88.100 80 {
        ! 健康检查的时间间隔
        delay_loop 6
        ! 负载均衡算法
        lb_algo wlc
        ! LVS模式，支持NAT/DR/TUN模式
        lb_kind DR
        protocol TCP
        nat_mask 255.255.255.0
        
        ! 真实Web服务器IP，端口
        real_server 192.168.88.10 80 {
            weight 3
            ! Web服务健康检查
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
               connect_timeout 1
            }
        }
        
        ! 真实Web服务器IP，端口
        real_server 192.168.88.11 80 {
            weight 3
            ! Web服务健康检查
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
               connect_timeout 1
            }
        }
    }

由于我们所有的服务器都在同一个子网中，因此无法使用**NAT**（Network Address Translation）模式，这里我们使用**DSR**（Direct Server Return）模式，使用DSR模式（配置时使用`DR`），需要web服务器将VIP绑定到自己的本地回环网卡上去，否则无法与web服务器通信。在 **node-1** 和 **node-2** 上，执行下面的命令

    ip addr add 192.168.88.100/32 dev lo

> 这里绑定的VIP只在本次设置有效，服务器重启后需要再次执行，因此，可以通过下面的方法永久的添加该IP
> 
> 在`/etc/sysconfig/network-scripts`目录下，创建`ifcfg-lo:0`文件
> 
>     DEVICE=lo:0
>     IPADDR=192.168.88.100
>     NETMASK=255.255.255.255
>     ONBOOT=yes
>     NAME=loopback
> 
> 然后重启网络服务（`systemctl restart network`）让其生效即可
    
接下来，重启两台 **keepalived** 服务器的服务就可以生效了

    systemctl restart keepalived

然后我们在客户端访问以下我们的web服务，这里我们就可以使用VIP来访问了

![-w267](https://oayrssjpa.qnssl.com/15362058058752.jpg)

可以看到，请求被分配到了两台真实的web服务器。在 **keepalived** 服务器上执行 `ipvsadm`

![-w565](https://oayrssjpa.qnssl.com/15362061774332.jpg)

如果此时**node-1**的服务挂了怎么办？我们来模拟一下，在**node-1**上面，我们停止web服务

    systemctl stop openresty 

等几秒之后（我们配置健康检查周期为6s）然后再来请求

![-w270](https://oayrssjpa.qnssl.com/15362060114525.jpg)

在 **keepalived** 服务器上执行 `ipvsadm`

![-w558](https://oayrssjpa.qnssl.com/15362061057170.jpg)

可以看到，有问题的 **node-1** 已经被剔除了。


### 扩展阅读

这里你可能会有两个疑问：

- 第一个是为什么无法使用NAT模式？

    使用NAT模式，正常的流程应该是这样的，在NAT模式下，客户端（**192.168.1.2**）请求经过负载均衡器（**192.168.88.100**）后，负载均衡器会修改目的IP地址为真实的web服务器IP地址 **192.168.88.10**，这样web服务器收到请求后，发现目的IP地址是自己，就可以处理该请求了。响应报文发送给负载均衡器，负载均衡器修改响应报文的源IP地址为自身VIP，这样客户端收到响应后就能够正常处理了。
    
    ![keepalive原理 -1-](https://oayrssjpa.qnssl.com/keepalive%E5%8E%9F%E7%90%86%20-1-.jpg)

    我们的客户端和服务器都在同一个子网下。处理完成后响应给客户端时，响应报文的源IP地址为 **192.168.88.10**，目的IP为 **192.168.88.2**，由于在同一个子网中，因此不会经过负载均衡器，而是直接将报文发送给了客户端。因此在响应报文中，源IP地址尚未经过修改直接发送给了客户端，导致无法正常完成通信。

    ![keepalive原理](https://oayrssjpa.qnssl.com/keepalive%E5%8E%9F%E7%90%86.jpg)

    
- 第二个是为什么使用DSR模式必须将VIP绑定到web服务器的网卡上去？

    在DSR模式下，发送给负载均衡器的报文没有经过任何修改就直接发送给了真实的web服务器，这时候目的IP地址是 VIP **192.168.88.100**，Web服务器收到该请求之后，发现目的IP地址不是自己，会认为这个报文不是发送给自己的，无法处理该请求。也就是说，在使用DSR模式下，仅仅在负载均衡器上做配置是无法实现负载均衡的。因此最简单的方式就是将VIP绑定到真实服务器的回环接口上。之所以子网掩码时 **255.255.255.255**（或者**/32**），是让其广播地址是其自身，避免其发送ARP到该子网的广播域，防止负载均衡器上的VIP和Web服务器的IP冲突。
    
    ![keepalive原理 -3-](https://oayrssjpa.qnssl.com/keepalive%E5%8E%9F%E7%90%86%20-3-.jpg)
    

对于负载均衡算法，我们这里采用了`wlc`（加权最小连接调度）。其它调度算法如下（图来自 《24小时365天不间断服务：服务器基础设施核心技术》一书）

![](https://oayrssjpa.qnssl.com/15362051776668.jpg)


## 总结

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

通过本文，相信你已经可以搭建一套高可用的Web服务了，Keepalived还有很多配置选项等待大家自己去发掘，我们不仅可以实现Web服务的高可用，还可以用来实现一些基础服务组件的高可用，比如MySQL、Redis、RabbitMQ等，本文也只是抛砖引玉了。

## 参考文献

- 24小时365天不间断服务：服务器基础设施核心技术

[vagrant]: https://www.vagrantup.com/
[vagrant-tutorial]: https://aicode.cc/vagrant-ru-men.html
