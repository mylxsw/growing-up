# Centos使用vsftp搭建FTP服务器

工作这么长时间了，这次竟然让搭建个FTP服务器给难住了，原本以为是分分钟搞定的事，结果折腾了一下午。

[TOC]

## 环境配置

### 安装 

    yum install -y vsftpd

### 启动、重启

    service vsftpd start
    service vsftpd restart

### 配置文件

编辑配置文件`/etc/vsftpd/vsftpd.conf`
    
    anonymous_enable=NO
    local_enable=YES
    write_enable=YES
    local_umask=022
    dirmessage_enable=YES
    xferlog_enable=YES
    connect_from_port_20=YES
    xferlog_std_format=YES
    ascii_upload_enable=YES
    ascii_download_enable=YES
    ftpd_banner=Welcome to Yunsom FTP service.
    ls_recurse_enable=YES
    listen=YES
    pam_service_name=vsftpd
    userlist_enable=YES
    tcp_wrappers=YES
    use_localtime=YES

### 新增用户

    useradd -d /data/axure --no-create-home -s /sbin/nologin axure
    passwd axure
    
    mkdir /data/axure
    chown axure:axure -R /data/axure

### 关闭SELinux/iptables

由于是用于内网使用，因此就简单粗暴一点吧

    setenforce 0
    service iptables stop


