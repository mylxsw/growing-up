# CentOS搭建samba服务器

[TOC]

## 环境搭建

### 安装

    yum install -y samba

### 配置

在 `/etc/samba/smb.conf` 最下面增加配置

    [yunsom]
        comment = yunsom
        path = /data/smb
        writable = yes


新增用户用于访问smb服务

    useradd --no-create-home --shell /sbin/nologin yunsom
    smbpasswd -a yunsom

设置存储目录权限

    chown -R yunsom:yunsom /data/smb


由于本次架设是在内网使用，简单粗暴起见，直接关闭selinux和iptables

    service iptables stop
    setenforce 0

> 如果希望彻底关闭selinux，可以编辑`/etc/selinux/config`，将`SELINUX`配置改为`SELINUX=disabled`。

### 启动、重启服务

    service smb start
    service smb restart

## 客户端连接

### Windows

![](https://oayrssjpa.qnssl.com/14925692897907.jpg)

![](https://oayrssjpa.qnssl.com/14925693294462.jpg)

![](https://oayrssjpa.qnssl.com/14925693594467.jpg)

### Mac连接

![](https://oayrssjpa.qnssl.com/14925694634863.jpg)

![](https://oayrssjpa.qnssl.com/14925694967324.jpg)

![](https://oayrssjpa.qnssl.com/14925695149364.jpg)

![](https://oayrssjpa.qnssl.com/14925695422387.jpg)

![](https://oayrssjpa.qnssl.com/14925695724625.jpg)



