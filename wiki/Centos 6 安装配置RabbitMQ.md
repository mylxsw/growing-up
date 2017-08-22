# Centos 6 安装配置RabbitMQ

[TOC]

## 安装

    wget https://bintray.com/rabbitmq/rabbitmq-server-rpm/download_file?file_path=rabbitmq-server-3.6.10-1.el6.noarch.rpm

    # 安装Erlang运行环境    
    rpm -ivh erlang-19.3.6-1.el6.x86_64.rpm
    # 安装依赖项目
    yum install socat -y
    # 安装rabbitmq服务
    rpm -ivh rabbitmq-server-3.6.10.rpm

## 配置开机自启动

    chkconfig rabbitmq-server on
    service rabbitmq-server start
    
## 配置管理界面

    rabbitmq-plugins enable rabbitmq_management
    
> 默认端口是 **15672**，初始用户guest只能在localhost访问。

## 使用配置

### 配置用户权限

    # 删除默认的guest用户
    rabbitmqctl delete_user guest
    # 新增用户mylxsw,密码aicode.cc
    rabbitmqctl add_user mylxsw aicode.cc
    
    # 查看当前的用户列表
    rabbitmqctl list_users
    # 查看用户有哪些权限
    rabbitmqctl list_user_permissions mylxsw
    
### 增加一个虚拟主机

    # 增加虚拟主机aicode
    rabbitmqctl add_vhost aicode
    # 列出有哪些虚拟主机
    rabbitmqctl list_vhosts
    
### 为用户授权虚拟主机的访问权限

    # 授权用户mylxsw访问aicode虚拟主机
    rabbitmqctl set_permissions -p aicode mylxsw ".*" ".*" ".*"
    
> 如果需要用户能够访问管理界面，需要设置用户为管理员 `rabbitmqctl set_user_tags mylxsw administrator`

### 启用tranc_on特性跟踪消息

只在开发调试问题的时候开启！

下面开启 `aicode` 虚拟主机的trace

    rabbitmqctl trace_on -p aicode

开启UI插件

    rabbitmq-plugins enable rabbitmq_tracing

![-w600](https://oayrssjpa.qnssl.com/15034157780454.jpg)

    
## Monit监控

    check process rabbitmq with pidfile /var/run/rabbitmq/pid
    	start program = "/etc/init.d/rabbitmq-server start"
    	stop program = "/etc/init.d/rabbitmq-server stop"
    	if failed port 5672 then restart
    	if 5 restarts within 5 cycles then timeout
    
## 错误解决

### 由于主机名是"数字.xxx"这种形式导致报错

主机名的第一位如果是数字的话，由于Rabbitmq的bug导致无法启动，报错如下

    epmd error for host 237: badarg (unknown POSIX error)

解决方案是修改`/etc/rabbitmq/rabbitmq-env.conf`，添加下面的配置

    HOSTNAME=rabbitmq

接下来在`/etc/hosts`中加入该主机名

    127.0.0.1   localhost rabbitmq


## 参考

- [Running rabbitmq on hosts with numeric hostnames](https://chr4.org/blog/2016/06/08/running-rabbitmq-on-hosts-with-numeric-hostnames/)

