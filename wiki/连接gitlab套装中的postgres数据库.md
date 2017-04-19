# 连接gitlab套装中的postgres数据库

有时候我们需要连接到gitlab的数据库执行一些批量的操作，比如批量替换所有项目的web hook。

[TOC]

## 连接

If you need to connect to the bundled PostgreSQL database and are using the default Omnibus GitLab database configuration, you can connect as the application user:

    sudo gitlab-rails dbconsole

or as a Postgres superuser:

    sudo gitlab-psql -d gitlabhq_production

## 批量更新webhook地址

    -- 更新web_hooks
    update web_hooks set url=replace(url,'192.168.1.223','deploy.aicode.cc') where url like 'http://192.168.1.223%';
    
    -- 查询更新结果
    select id, url from web_hooks where url like 'http://deploy.aicode.cc%';

## 参考

- [Connecting to the bundled PostgreSQL database ](http://docs.gitlab.com/omnibus/settings/database.html#connecting-to-the-bundled-postgresql-database)

