# MySQL进阶：INFORMATION_SCHEMA 简介

[TOC]

在使用命令行连接 MySQL 的时候，我们在执行 `SHOW DATABASES` 命令时，会发现除了自己拥有权限的数据库之外，还有另外一个名为 **INFORMATION_SCHEMA** 的表，这个表示用来做什么用的呢？

![-w677](media/15723283045009/15723434564572.jpg)


在 MySQL 中， **INFORMATION_SCHEMA** 是用来访问数据库的元数据（比如数据库，表的名称，列的数据类型或者访问权限等）的，在每个 MySQL 的实例中，**INFORMATION_SCHEMA** 保存了它维护的所有数据库的信息，这个库中包含了很多只读的表（它们实际上是一些视图，因此并没有与之关联的文件，你可以无法为他们创建触发器），用于满足对 MySQL 服务本身的不同查询需求。

> 你可以通过 `USE` 语句选择使用 **INFORMATION_SCHEMA** 作为默认的数据库，但是只能对其执行读取操作，无法执行 `INSERT`，`UPDATE` 和 `DELETE` 操作。

比如，下面的的 SQL 可以查询出数据库 *wizard* 中所有的表以及数据类型，存储引擎。

    mysql> SELECT table_name, table_type, engine
        -> FROM information_schema.tables
        -> WHERE table_schema = 'wizard'
        -> ORDER BY table_name;
    +----------------------+------------+---------+
    | table_name           | table_type | engine  |
    +----------------------+------------+---------+
    | migrations           | BASE TABLE | InnoDB  |
    | notifications        | BASE TABLE | InnoDB  |
    | wz_attachments       | BASE TABLE | InnoDB  |
    | wz_categories        | BASE TABLE | InnoDB  |
    | wz_comments          | BASE TABLE | InnoDB  |
    | wz_groups            | BASE TABLE | InnoDB  |
    | wz_operation_logs    | BASE TABLE | ARCHIVE |
    | wz_pages             | BASE TABLE | InnoDB  |
    | wz_page_histories    | BASE TABLE | InnoDB  |
    | wz_page_share        | BASE TABLE | InnoDB  |
    | wz_page_tag          | BASE TABLE | InnoDB  |
    | wz_password_resets   | BASE TABLE | InnoDB  |
    | wz_projects          | BASE TABLE | InnoDB  |
    | wz_project_catalogs  | BASE TABLE | InnoDB  |
    | wz_project_group_ref | BASE TABLE | InnoDB  |
    | wz_project_stars     | BASE TABLE | InnoDB  |
    | wz_tags              | BASE TABLE | InnoDB  |
    | wz_templates         | BASE TABLE | InnoDB  |
    | wz_users             | BASE TABLE | InnoDB  |
    | wz_user_group_ref    | BASE TABLE | InnoDB  |
    +----------------------+------------+---------+
    20 rows in set (0.00 sec)

在 MySQL 中，每个用户都有对 **INFORMATION_SCHEMA** 的访问权限，但是只能看到表中他们有权限的对象的信息，也有点些场景下用户如果没有权限，看到的是 NULL。对于 [InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html) 表来说，必须拥有 [PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process) 权限才能查看。

由于使用 **INFORMATION_SCHEMA** 查询可能会从多个数据库检索信息，所以查询可能会比较耗时，对性能产生一定的影响。在执行之前，可以使用 `EXPLAIN` 命令检查一下查询的效率，关于如何优化 **INFORMATION_SCHEMA** 查询效率，参考 [Optimizing INFORMATION_SCHEMA Queries](https://dev.mysql.com/doc/refman/8.0/en/information-schema-optimization.html)。

## 不同表的用途

在不同版本的 MySQL/MariaDB 中， **INFORMATION_SCHEMA** 中的表并不完全一样，但是大部分都是一致的，下面是 MariaDB 10.3 中包含的表，我对它们一一做了注释

| 表名 | 用途 |
|----|----|
| ALL_PLUGINS                           | 服务器所有插件的信息，无论是否已经安装 |
| PLUGINS                               | 服务器安装的插件信息 |
| APPLICABLE_ROLES                      | 当前用户可以使用的角色信息 |
| CHARACTER_SETS                        | 可用的字符集信息 |
| CHECK_CONSTRAINTS                     | 表上定义的 CHECK 约束信息 |
| COLLATIONS                            | 字符集排序规则信息 |
| COLLATION_CHARACTER_SET_APPLICABILITY | 字符集和排序规则的对应关系 |
| COLUMNS                               | 表中的列信息 |
| COLUMN_PRIVILEGES                     | 列的权限信息，数据来源于 `mysql.columns_priv` 系统表 |
| ENABLED_ROLES                         | 当前会话的角色信息 |
| ENGINES                               | 存储引擎的信息，可以用于检查引擎是否支持 |
| EVENTS                                | 关于事件管理器的事件信息 |
| FILES                                 | 表空间数据存储文件的信息 |
| GLOBAL_STATUS                         | 所有的状态变量值，对应命令 `SHOW GLOBAL STATUS` |
| GLOBAL_VARIABLES                      | 所有的系统变量值，对应命令 `SHOW GLOBAL VARIABLES` |
| SESSION_STATUS                        | 所有的会话的状态变量值，对应命令 `SHOW SESSION STATUS` |
| SESSION_VARIABLES                     | 所有的会话变量，对应命令 `SHOW SESSION VARIABLES` |
| KEY_CACHES                            | 关于 [Segmented Key Cache][mariadb-segmented-key-cache] 的统计信息 |
| KEY_COLUMN_USAGE                      | 描述了索引列有哪些约束 |
| PARAMETERS                            | 存储过程参数，返回值信息 |
| PARTITIONS                            | 表分区信息，没一行对应了一个独立的分区或者分区表的子分区 |
| PROCESSLIST                           | 提供了哪些线程正在运行的信息 |
| PROFILING                             | 提供了语句剖析信息，它的内容对应了 SHOW PROFILE 和 SHOW PROFILES 语句的信息 |
| REFERENTIAL_CONSTRAINTS               | 外键信息 |
| ROUTINES                              | 存储过程信息 |
| SCHEMATA                              | 数据库的信息 |
| SCHEMA_PRIVILEGES                     | 数据库权限信息，数据来源于 `mysql.db` 系统表 |
| STATISTICS                            | 表索引信息 |
| SYSTEM_VARIABLES                      | 所有系统变量当前的值和各种元数据 |
| TABLES                                | 表的信息 |
| TABLESPACES                           | MySQL 集群的表空间信息 |
| TABLE_CONSTRAINTS                     | 描述了哪个表有约束 |
| TABLE_PRIVILEGES                      | 表权限信息，数据来源于 `mysql.table_priv` 系统表 |
| TRIGGERS                              | 关于触发器的信息，必须有表的 `TRIGGER` 权限才能查看 |
| USER_PRIVILEGES                       | 全局权限信息，数据来源于 `mysql.user` 系统表 |
| VIEWS                                 | 数据库视图信息 |
| GEOMETRY_COLUMNS                      | 表中存储空间数据的列的信息 |
| SPATIAL_REF_SYS                       | 存储了数据库中使用的每个空间参考系统的信息 |
| CLIENT_STATISTICS                     | 客户端连接的统计信息，作为 [用户统计][user-statistics] 特性的一部分，默认不开启 |
| USER_STATISTICS                       | 用户活动的统计信息，作为 [用户统计][user-statistics] 特性的一部分，默认不开启 |
| INDEX_STATISTICS                      | 索引使用统计，用于定位未使用的索引以及生成删除命令，作为 [用户统计][user-statistics] 特性的一部分，默认不开启 |
| TABLE_STATISTICS                      | 表使用的统计信息，作为 [用户统计][user-statistics] 特性的一部分，默认不开启 |

在所有的存储引擎中，我们最常用的就是 InnoDB 存储引擎了，下面是 InnoDB 相关的表

| 表名 | 用途 |
|----|----|
| INNODB_SYS_DATAFILES                  | 数据文件存储路径信息 |
| INNODB_SYS_TABLESTATS                 | 状态信息，可以用于开发性能相关的扩展或者高级的性能监控 |
| INNODB_SYS_FIELDS                     | 索引的字段信息 |
| INNODB_SYS_COLUMNS                    | 字段信息 |
| INNODB_SYS_FOREIGN_COLS               | 外键列的信息 |
| INNODB_SYS_FOREIGN                    | 外键信息 |
| INNODB_SYS_TABLES                     | 表信息 |
| INNODB_SYS_TABLESPACES                | 表空间信息 |
| INNODB_SYS_INDEXES                    | 索引信息 |
| INNODB_SYS_VIRTUAL                    | 虚拟列的元信息 |
| INNODB_SYS_SEMAPHORE_WAITS            | 当前的信号量等待信息 |
| INNODB_TABLESPACES_SCRUBBING          | 关于 [数据清理][data-scrubbing] 的信息 |
| INNODB_CMPMEM                         | 缓冲池中压缩页的信息，可用于度量表压缩效率 |
| INNODB_CMPMEM_RESET                   | 同 `INNODB_CMPEM`，但是每次查询这个表会清空 `RELOCATION_TIME` 字段的值 |
| INNODB_CMP_PER_INDEX                  | 包含了以独立的索引分组的与压缩操作相关的状态信息 |
| INNODB_CMP_PER_INDEX_RESET            | 同 `INNODB_CMP_PER_INDEX`， 但是每次查询之后都会清空数据 |
| INNODB_CMP                            | 包含了与压缩操作相关的状态信息 |
| INNODB_CMP_RESET                      | 同 `INNODB_CMP`，但是每次查询之后会清空数据 |
| [INNODB_LOCK_WAITS][innodb_lock_waits] | 阻塞的事务信息 |
| INNODB_TABLESPACES_ENCRYPTION         | 加密的表空间信息 |
| INNODB_BUFFER_PAGE_LRU                | 有关缓冲池中页的信息，以及出于清除目的如何对页进行排序 |
| INNODB_BUFFER_PAGE                    | 缓冲池中页的信息 |
| INNODB_BUFFER_POOL_STATS              | 缓冲池中页的信息，与 `SHOW ENGINE INNODB STATUS` 语句返回的内容类似 |
| INNODB_FT_INDEX_TABLE                 | 全文索引信息 |
| INNODB_FT_DELETED                     | 包含了从全文索引中已经删除的行，这些信息用于过滤查询请求的结果，解决每次删除一行时昂贵的重新组织索引操作 |
| INNODB_FT_INDEX_CACHE                 | 最近插入到全文索引的行信息，为了避免每次改变都去重新组织索引，新的变更只在 `OPTIMIZE TABLE` 命令运行之后才会合并到全文索引 |
| INNODB_FT_BEING_DELETED               | 当 `OPTIMIZE TABLE` 正在执行中，此时发生了与 `INNODB_FT_DELETED` 有关的文档 |
| INNODB_FT_DEFAULT_STOPWORD            | 包含了用于创建全文索引的[停止词列表](stopwords) |
| INNODB_FT_CONFIG                      | 全文索引的元数据 |
| [INNODB_TRX][innodb_trx]              | 所有当前正在执行的事务的信息 |
| [INNODB_LOCKS][innodb_locks]          | 包含了事务请求但是未获得的锁或者阻塞其它事务的锁的信息 |
| INNODB_METRICS                        | 一些有用的性能指标 |
| INNODB_MUTEXES                        | 监控互斥锁和读写锁 |

## 总结 

本文只是对 **INFORMATION_SCHEMA** 数据库是什么，以及都有哪些表以及它们的用途做了个简要的概述，在了解这个数据库的基础之后，我们在[下篇文章][part-2]中将会详细介绍 [事务，锁相关表以及如何排查死锁问题][part-2]，敬请关注。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 参考文档

- [Mariadb Knowledge Base: Information Schema Tables](https://mariadb.com/kb/en/library/information-schema-tables/)
- [MySQL 8.0 Reference Manual: INFORMATION_SCHEMA Tables](https://dev.mysql.com/doc/refman/8.0/en/information-schema.html)


[mariadb-segmented-key-cache]:https://mariadb.com/kb/en/library/segmented-key-cache/
[user-statistics]:https://mariadb.com/kb/en/library/user-statistics/
[innodb_locks]:https://mariadb.com/kb/en/library/information-schema-innodb_locks-table/
[innodb_lock_waits]:https://mariadb.com/kb/en/library/information-schema-innodb_lock_waits-table/
[data-scrubbing]:https://mariadb.com/kb/en/library/innodb-data-scrubbing/
[stopwords]:https://mariadb.com/kb/en/stopwords/
[innodb_trx]:https://mariadb.com/kb/en/library/information-schema-innodb_trx-table/
[part-2]:https://github.com/mylxsw/growing-up/blob/master/doc/mysql-lock-transaction.md
