# 配置Redis主从服务

[TOC]

我们在两台服务器上，组成一个master-slave的结构Redis服务。Redis我们使用 **4.0.6** 版本。

| SERVER_IP | USAGE |
| --- | --- |
| 192.168.200.11 | MASTER |
| 192.168.200.12 | SLAVE |

Redis访问密码我们设置为 **1234568**，基本目录结构统一如下

| DIRECTORY | USAGE |
| --- | --- |
| /data/services/redis-data/6379 | Redis数据存储 |
| /data/logs/redis/redis_6379.log | Redis日志文件 |


## 单机安装配置

### 安装

以下安装步骤需要在 **两台** 服务器上分别实施。

首先下载最新版的Redis服务

    wget http://download.redis.io/releases/redis-4.0.6.tar.gz

下载之后解压，然后执行编译安装

    tar -zxvf redis-4.0.6.tar.gz
    cd redis-4.0.6
    
    # 编译安装
    make
    make install
    
    # 安装配置
    cd utils
    ./install_server.sh
    
    # 关闭服务
    /etc/init.d/redis_6379 stop

在执行 **./install_server.sh** 命令的时候，我们做以下配置

    Port           : 6379
    Config file    : /etc/redis/6379.conf
    Log file       : /data/logs/redis/redis-6379.log
    Data dir       : /data/services/redis-data/.6379
    Executable     : /usr/local/bin/redis-server
    Cli Executable : /usr/local/bin/redis-cli


### 配置

修改 **/etc/redis/6379.conf** 配置文件

- **192.168.200.11**

        bind 192.168.200.11 127.0.0.1
        requirepass 1234568

- **192.168.200.12**

        bind 192.168.200.12 127.0.0.1
        requirepass 1234568


## 配置主从

主从配置只需要在从(slave)上进行配置即可，主(master)不需要做配置，因此，这里就只需要配置 **192.168.200.12** 了

修改 **192.168.200.12** 的 **/etc/redis/6379.conf**

    slaveof 192.168.200.11 6379
    masterauth 1234568


## 运行

配置完成后，就可以启动两台redis服务了

    /etc/init.d/redis_6379 start

启动之后，使用 **redis-cli** 分别连接到两台服务器执行 `info` 命令，可以看到服务状态

    $ redis-cli -h 192.168.200.11 -a 1234568
    192.168.200.11:6379> info
    ...
    role:master
    connected_slaves:1
    slave0:ip=192.168.200.12,port=6379,state=online,offset=2128,lag=0
    master_replid:0122a2d06ca5c5c7383d059e27bca0da266727ce
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:2128
    second_repl_offset:-1
    ...
    
    $ redis-cli -h 192.168.200.12 -a 1234568
    192.168.200.12:6379> info
    ...
    role:slave
    master_host:192.168.200.11
    master_port:6379
    master_link_status:up
    master_last_io_seconds_ago:7
    master_sync_in_progress:0
    slave_repl_offset:2338
    slave_priority:100
    slave_read_only:1
    connected_slaves:0
    master_replid:0122a2d06ca5c5c7383d059e27bca0da266727ce
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:2338
    second_repl_offset:-1
    ...

看到上面这些，我们的Redis主从服务就搭建完毕了，其中主节点是读写的，从节点只读。

## 配置宕机后自动重启

当服务宕机后，自然要自动重启了啦，我们使用Linux系统的 **monit** 就好了

在 **/etc/monitrc** 中配置

    check process redis_6379 with pidfile "/var/run/redis_6379.pid"
        start program = "/etc/init.d/redis_6379 start"
        stop program = "/etc/init.d/redis_6379 stop"

之后，重新加载 **monit** 的配置就可以了

    monit reload
    
    
~


