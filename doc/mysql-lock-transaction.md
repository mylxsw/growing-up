# MySQL进阶：事务、锁相关表以及死锁问题

[TOC]

## 事务、锁相关表详解

对于每个表中包含了哪些字段，以及它们之间的关系，这里就不做详细展开，这里只重点说一下平时排查问题时常用的四个表

- PROCESSLIST
- INNODB_TRX
- INNODB_LOCKS
- INNODB_LOCK_WAITS

对其他表的使用，感兴趣或者用到的可以根据需要查文档。

### PROCESSLIST

**PROCESSLIST** 表包含了运行中的线程信息。

| 列名 | 说明 |
| ------ | ------ |
| ID | 连接标识符 |
| USER | 数据库用户 |
| HOST | 用户的主机地址 |
| DB | 默认数据库，如果没有则为 `NULL` |
| COMMAND | 运行中的命令类型，取值参考 [Thread Command Values][thread-command-values] |
| TIME | 线程处于当前状态的时间，单位是秒 |
| STATE | 线程当前的状态，参考 [Thread States][thread-states] |
| INFO | 线程正在执行的语句，如果没有则为 `NULL` |
| TIME_MS | 线程处于当前状态的时间(以毫秒为单位，精度为微秒) |
| STAGE | 该过程当前所处的阶段 |
| MAX_STAGE | 最大阶段的数 |
| PROGRESS | 当前阶段内的过程进度(0-100%) |
| MEMORY_USED | 线程使用的内存，单位是字节 |
| EXAMINED_ROWS | 线程检查的行数，只对 UPDATE, DELETE 以及相似的命令有效，对 SELECT 命令等这个值为 0 |
| QUERY_ID | 查询 ID |
| INFO_BINARY | 二进制的数据信息 |

其中，COMMAND 类型的取值参看下面的表格

| 值 | 说明 |
| ----- | ------- |
| Binlog Dump | 主线程发送 binlog 到从库 |
| Change user | 执行改变用户操作|
| Close stmt | 正在关闭 [Prepared statement][prepared-statements] |
| Connect |  从库已经连接到主库 |
| Connect Out | 从库正在连接主库 |
| Create DB | 执行创建数据库的操作 |
| Daemon | 内部服务线程，非客户端连接 |
| Debug | 正在生成调试信息 |
| Delayed insert | 延迟插入处理器 |
| Drop DB | 执行 DROP 数据库操作 |
| Error | 错误 |
| Execute | 正在执行 [Prepared statement][prepared-statements] |
| Fetch | 正在获取已经执行完成的 [Prepared statement][prepared-statements] 的结果 |
| Field List | 检索表的列信息 |
| Init DB | 正在选择默认数据库 |
| Kill | 正在 Kill 其它线程 |
| Long Data | 正在从 [Prepared statement][prepared-statements] 的结果中加载很长的数据 |
| Ping | 服务器 Ping 请求处理器 |
| Prepare | 准备 [Prepared statement][prepared-statements] |
| Processlist | 正在准备关于服务器线程的 processlist 信息 |
| Query | 正在执行语句 |
| Quit | 线程正在终止 |
| Refresh |  Flush 表、日志、缓存，或者刷新副本服务器或者状态变量信息 |
| Register Slave | 正在注册从库 |
| Reset stmt | 正在重置 [Prepared statement][prepared-statements] |
| Set option | 正在设置或者重置客户端执行语句的选项 |
| Sleep | 等待客户端发送新的语句 |
| Shutdown | 正在关闭服务器 |
| Statistics | 正在准备关于服务器的状态信息 |
| Table Dump | 正在发送表的内容给从库 |
| Time | 没有使用 |


### INNODB_TRX

**INNODB_TRX** 表存储了当前正在执行的所有事务信息，下面是每一列的说明

| 列名 | 说明 |
| ------ | ------ |
| TRX_ID | 唯一的事务 ID |
| TRX_STATE | 事务的执行状态： `RUNNING`, `LOCK WAIT`, `ROLLING BACK`, `COMMITTING` |
| TRX_STARTED | 事务开始时间 |
| TRX_REQUESTED_LOCK_ID | 如果 `TRX_STATE`=`LOCK_WAIT`, 则为正在等待的锁的 [INNODB_LOCKS.LOCK_ID][innodb_lock_id]；其他状态为 `NULL` |
| TRX_WAIT_STARTED | 如果 `TRX_STATE`=`LOCK_WAIT`, 则为事务开始等待锁的时间；其它状态为 `NULL`. |
| TRX_WEIGHT | 基于锁定的行数和更改的行数得出的事务权重，要解决死锁，首先会回滚低权重的事务。如果事务中包含了非事务表，则该事务拥有更高的权重 |
| TRX_MYSQL_THREAD_ID | 表 [PROCESSLIST][processlist] 中的线程 ID（注意的是，锁和事务信息表使用的是 processlist 的快照，因此这两者中的记录可能是不同的） |
| TRX_QUERY | 当前事务正在执行的 SQL 语句 |
| TRX_OPERATION_STATE | 当前事务的状态或者 `NULL` |
| TRX_TABLES_IN_USE | 当前 SQL 语句所使用的 InnoDB 表的数量 |
| TRX_TABLES_LOCKED | 当前 SQL 语句持有行锁的 InnoDB 表的数量 |
| TRX_LOCK_STRUCTS | 事务保留的锁的数量 |
| TRX_LOCK_MEMORY_BYTES | 用于保存当前事务的锁的结构体总大小（字节） |
| TRX_ROWS_LOCKED | 当前事务锁的数据行数，这是一个近似值，可能包含了对当前事务不可见的行（这些行已经标记为删除了，但是物理位置上还存在）|
| TRX_ROWS_MODIFIED | 当前事务新增或者修改的行数 |
| TRX_CONCURRENCY_TICKETS | 标识出当前事务在被换出之前还能做多少工作 ，详情参考 [innodb_concurrency_tickets][innodb_concurrency_tickets] 系统变量 |
| TRX_ISOLATION_LEVEL | [当前事务的隔离级别][trx_iso_level] |
| TRX_UNIQUE_CHECKS | 当前事务的唯一性检查是否开启 |
| TRX_FOREIGN_KEY_CHECKS | 外键约束检查是否开启 |
| TRX_LAST_FOREIGN_KEY_ERROR | 最后一次外键错误，没有的话为 `NULL` |
| TRX_ADAPTIVE_HASH_LATCHED | 😒 I don't know what this means |
| TRX_ADAPTIVE_HASH_TIMEOUT | 😒 I don't know what this means |
| TRX_IS_READ_ONLY | 只读事务为 `1`，否则为 `0` |
| TRX_AUTOCOMMIT_NON_LOCKING | 如果事务只包含了一个语句，则为 `1`（一个没有使用 `FOR UPDATE` 或者 `LOCK IN SHARED MODE` 的 `SELECT` 语句，并且 autocommit 是开启的）；如果这个值和 `TRX_IS_READ_ONLY` 同时为 `1`，则事务可以由存储引擎优化以减少一些开销 |

### INNODB_LOCKS

> 这个表在 MySQL 5.7.14 中已经弃用，并且在 MySQL 8.0 中已经移除了（使用 performance_schema.data_lock_waits 代替，[下篇文章][grow-up-mysql-performance_schema] 将会讲解 ）。

| 列名 | 说明 |
|----|----|
| LOCK_ID | 锁的 ID，格式并不固定  |
| LOCK_TRX_ID | 持有锁的事务 ID，与 INNODB_TRX.TRX_ID 对应 |
| LOCK_MODE | [锁模式][lock-mode]： S (共享锁), X (排它锁), IS (意向共享锁), IX (意向排它锁), S_GAP (共享间隙锁), X_GAP (排他间隙锁), IS_GAP (意向共享间隙锁), IX_GAP (意向排他间隙锁), AUTO_INC (自增的表级锁) |
| LOCK_TYPE | 锁的类型：RECORD 或者 TABLE |
| LOCK_TABLE | 被锁的表或者包含被锁的行的表 |
| LOCK_SPACE | 如果 LOCK_TYPE=RECORD，为表空间 ID，否则为 `NULL` |
| LOCK_INDEX | 如果 LOCK_TYPE=RECORD，为索引名，否则为 `NULL` |
| LOCK_PAGE | 如果 LOCK_TYPE=RECORD，为被锁的记录所在的页号，否则为 `NULL` |
| LOCK_REC | 如果 LOCK_TYPE=RECORD，为被锁的记录的堆号，否则为 `NULL` |
| LOCK_DATA | 如果 LOCK_TYPE=RECORD，为被锁的记录的主键（作为 SQL 字符串），否则为 `NULL`。如果没有主键，则使用 InnoDB 内部的 row_id。为了避免不必要的 IO，如果被锁记录的页不在缓冲池中，则个字段也为 `NULL` |

### INNODB_LOCK_WAITS

**INNODB_LOCK_WAITS** 表包含了关于阻塞的 InnoDB 事务的信息。

> 这个表在 MySQL 5.7.14 中已经弃用，并且在 MySQL 8.0 中已经移除了（使用 performance_schema.data_lock_waits 代替，[下篇文章][grow-up-mysql-performance_schema] 将会讲解 ）。

| 列名 | 说明 |
|----|----|
| REQUESTING_TRX_ID | 正在请求（阻塞）的事务 ID |
| REQUESTED_LOCK_ID | 事务正在等待的锁的 ID，详情可以关联 `INNODB_LOCKS`.`LOCK_ID` 来查询 |
| BLOCKING_TRX_ID | 阻塞中的事务的 ID |
| BLOCKING_LOCK_ID | 正在阻塞其它事务的事务所持有的锁 ID，关于这个锁的详情，关联 `INNODB_LOCKS` 表的 `LOCK_ID` 字段 |

## 死锁问题


## 总结

## 参考文档

- [Mariadb Knowledge Base: Information Schema Tables](https://mariadb.com/kb/en/library/information-schema-tables/)
- [MySQL 8.0 Reference Manual: INFORMATION_SCHEMA Tables](https://dev.mysql.com/doc/refman/8.0/en/information-schema.html)


[innodb_lock_id]:https://mariadb.com/kb/en/information-schema-innodb_locks-table/
[processlist]:https://mariadb.com/kb/en/information-schema-processlist-table/
[innodb_concurrency_tickets]:https://mariadb.com/kb/en/xtradbinnodb-server-system-variables/#innodb_concurrency_tickets
[trx_iso_level]:https://mariadb.com/kb/en/set-transaction/#isolation-levels
[lock-mode]:https://mariadb.com/kb/en/xtradbinnodb-lock-modes/
[data_lock_waits]:https://dev.mysql.com/doc/refman/8.0/en/data-lock-waits-table.html
[thread-command-values]:https://mariadb.com/kb/en/library/thread-command-values/
[thread-states]:https://mariadb.com/kb/en/library/thread-states/
[prepared-statements]:https://mariadb.com/kb/en/prepared-statements/

[grow-up-mysql-performance_schema]: https://github.com/mylxsw/growing-up/blob/master/doc/mysql-performance_schema.md
