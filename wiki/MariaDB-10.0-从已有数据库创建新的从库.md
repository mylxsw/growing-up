# MariaDB 10.0 从已有数据库创建新的从库

[TOC]

## 备份

已有主库需要持续为用户提供服务，因此不能够停机或者重启，所以需要采用热备份的方式创建一个当前数据库的副本。

    innobackupex --user=root --password=PASSWORD --no-timestamp /data/backup/20190314/
	
> **innobackupex** 实际上是个perl脚本，封装了 **xtrabackup** 程序的使用，安装执行：`yum install -y percona-xtrabackup`


## 传输到从库服务器

备份完成后，打包传输到从库所在服务器

    tar -zcvf 20190314.tar.gz ./20190314
    scp 20190314.tar.gz root@xx.xx.xx.xx:/data

在从库所在服务器(xx.xx.xx.xx)上面，解压该压缩包

    cd /data
    tar -zxvf 20190314.tar.gz 

## 准备恢复备份

    innobackupex --apply-log ./20190314

![Xnip2019-03-14_17-36-33-w704](https://ssl.aicode.cc/Xnip2019-03-14_17-36-33.jpg)

注意图中红框中的内容，这部分内容非常关键，记录了当前的binlog文件名称和偏移量。后面我们创建主从关系的时候需要用到，当前文件名为 `mysql-bin.000001`，偏移量为 `369472581`。

## 恢复备份文件

    innobackupex --copy-back ./20190314

> 该命令会根据mariadb配置文件 my.cnf，将备份文件还原到mariadb数据目录，比如 /data/mysql

![Xnip2019-03-14_17-39-14-w835](https://ssl.aicode.cc/Xnip2019-03-14_17-39-14.jpg)

根据数据库的大小，经过漫长的等待，都是类似的文件拷贝...

![Xnip2019-03-14_17-49-24-w835](https://ssl.aicode.cc/Xnip2019-03-14_17-49-24.jpg)


执行备份恢复之后，需要修复文件权限

    chown -R mysql:mysql /data/mysql

## 重启从库

恢复完成后，启动mariadb

    systemctl start mysql

登录到mariadb

    mysql -uroot -p

## 建立主从关系

创建主从同步

    mysql> CHANGE MASTER TO MASTER_HOST='master服务器ID', 
        MASTER_USER='复制用户', 
        MASTER_PASSWORD='复制用户密码', 
        MASTER_LOG_FILE='mysql-bin.000001', 
        MASTER_LOG_POS=369472581;
    mysql> START SLAVE;

最后，查看主从同步状态

    mysql> SHOW SLAVE STATUS\G;

![Xnip2019-03-14_18-01-02-w826](https://ssl.aicode.cc/Xnip2019-03-14_18-01-02.jpg)

