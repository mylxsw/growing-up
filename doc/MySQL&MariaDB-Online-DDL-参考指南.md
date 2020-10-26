
[TOC]

![图文无关](https://ssl.aicode.cc/prometheus/20201026115911.JPG)

## 概述

在早期的 MySQL 版本中，DDL 操作（如创建索引等）通常都需要对数据表加锁，操作过程中 DML 操作都会被阻塞，影响正常业务。MySQL 5.6 和 MariaDB 10.0 开始支持  Online  DDL，可以在执行 DDL 操作的同时，不影响 DML 的正常执行，线上直接执行 DDL 操作对用户基本无感知（部分操作对性能有影响）。

不同版本的数据库对各种 DDL 语句的支持存在一定的差异，本文将会针对 MySQL 和 MariaDB 对 Online DDL 的支持情况做一个汇总，在需要执行 DDL 操作时，可以参考本文的 *Online DDL 支持情况* 部分。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

在 `ALTER TABLE` 语句中，支持通过 `ALGORITHM` 和 `LOCK` 语句来实现 Online  DDL：

- `ALGORITHM` -  控制 DDL 操作如何执行，使用哪个算法
- `LOCK` - 控制在执行 DDL 时允许对表加锁的级别

```sql
ALTER TABLE tab ADD COLUMN c varchar(50), ALGORITHM=INPLACE, LOCK=NONE;
```

### ALGORITHM 支持的算法

| ALGORITHM | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| DEFAULT   | 默认算法，自动使用可用的最高效的算法                         |
| COPY      | 最原始的方式，所有的存储引擎都支持，不使用 Online DDL，操作时会创建临时表，执行全表拷贝和重建，过程中会写入 Redo Log 和大量的 Undo Log，需要添加读锁，非常低效 |
| INPLACE   | 尽可能避免表拷贝和重建，更确切的名字应该是 `ENGINE` 算法，**由存储引擎决定如何实现**，有些操作是可以立即生效的（比如重命名列，改变列的默认值等），但有些操作依然需要全表或者部分表的拷贝和重建（比如添加删除列、添加主键、改变列为 NULL 等） |
| NOCOPY    | 该算法是 `INPLACE` 算法的子集，用于**避免聚簇索引（主键索引）的重建造成全表重建**，也就说用该算法会**禁止任何引起聚簇索引重建的操作** |
| INSTANT   | 用于避免 `INPLACE` 算法在需要修改数据文件时异常低效的问题，**所有涉及到表拷贝和重建的操作都会被禁止** |

> `NOCOPY` 算法支持：**MariaDB 10.3.2**+，**MySQL 不支持该算法**。 
>
> `INSTANT` 算法支持：**MariaDB 10.3.2**+，**MySQL 8.0.12**+。

算法使用规则：

- 如果用户指定的算法为 `COPY`，则 InnoDB 使用 `COPY` 算法。
- 如果用户指定的是 `COPY` 之外的其它算法，则 InnoDB 会按照算法效率，选择最高效的算法，最差的情况下采用用户指定的算法。比如用户指定了 `ALOGRITHM = NOCOPY`，则 InnoDB 会从 (NOCOPY, INSTANT) 中选择支持的最高效的算法。

![ALGORITHM 优劣](https://ssl.aicode.cc/prometheus/20201012160311.png)

MySQL 服务主要为 **Server 层** 和 **存储引擎层** 两部分组成，Server 层包含了 MySQL 大部分核心功能，所有的内置函数，跨存储引擎的功能如存储过程、触发器、视图等。存储引擎层负责数据的存储和读取，采用了插件式的架构模式。

**COPY 算法** 作用在 Server 层，其执行过程都是在 Server 层，因此所有存储引擎都支持使用该算法，执行过程如下图

![COPY算法执行过程](https://ssl.aicode.cc/prometheus/20201013175400.png)

**INPLACE 算法** 作用于存储引擎层，是 InnoDB 存储引擎特有的 DDL 算法，执行过程如下图所示

![INPLACE 算法执行过程](https://ssl.aicode.cc/prometheus/20201019175754.png)

### LOCK 策略

默认情况下，MySQL/MariaDB 在执行 DDL 期间会使用尽可能少的锁，如果必要，可以通过 LOCK 子句控制在执行 DDL 时允许对表加锁的级别。如果指定的操作所要求的限制级别不满足（**EXCLUSIVE > SHARED > NONE**），则语句执行失败并报错。

| 策略      | 说明                                          |
| --------- | --------------------------------------------- |
| DEFAULT   | 使用当前操作支持的粒度最小的锁策略            |
| NONE      | 不获取任何表锁，允许所有的 DML 操作           |
| SHARED    | 对表添加共享锁（读锁），只允许只读的 DML 操作 |
| EXCLUSIVE | 对表添加排它锁（写锁），不允许任何 DML 操作   |

> 为了避免执行 DDL 时，由于锁表导致生产服务不可用，在执行表结构变更语句时，可以添加 `LOCK=NONE` 子句，如果语句需要获取共享锁或者排它锁，则会直接报错，这样就可以避免意外锁表，造成线上服务不可用了。

## Online DDL 执行过程

 Online  DDL 操作主要分为三个阶段：

![Online DDL 执行过程](https://ssl.aicode.cc/prometheus/20201019180202.png)

- **阶段 1**：初始化

  在初始化阶段，服务器会根据存储引擎的能力，操作的语句和用户指定的 `ALGORITHM` 和 `LOCK` 选项来决定允许多大程度的并发。在这个阶段会创建一个 **可升级的元数据共享锁**（SU）来保护表定义。

- **阶段 2**：执行

  这个阶段会 **准备** 并 **执行** DDL 语句，根据 **阶段 1** 评估的结果来决定是否将元数据锁升级为 **排它锁** （X），如果需要升级为排它锁，则只在 DDL 的 **准备阶段** 短暂的添加排它锁。

- **阶段 3**：提交表定义

  在表定义的提交阶段，元数据锁会升级为排它锁来更新表的定义。独占排它锁的持续时间非常短。

> 元数据锁（MDL，Metadata Lock）主要用于 DDL 和 DML 操作之间的并发访问控制，保护表结构（表定义）的一致，保证读写的正确性。MDL 不需要显式的使用，在访问表时会自动加上。
>
> ![MDL](https://ssl.aicode.cc/prometheus/20201019135445.png)

由于上面三个阶段中对元数据锁的独占，  Online  DDL 过程必须等待已经持有元数据锁的并发事务提交或者回滚才能继续执行。

> 注意：当  Online  DDL 操作正在等待元数据锁时，该元数据锁会处于挂起状态，**后续的所有事务都会被阻塞**。在 MariaDB 10.3 之后，可以通过添加 `NO WAIT` 或者 `WAIT n` 来控制等待所得超时时间，超时立即失败。
>
> ```sql
> ALTER TABLE tbl_name [WAIT n|NOWAIT] ...
> CREATE ... INDEX ON tbl_name (index_col_name, ...) [WAIT n|NOWAIT] ...
> DROP INDEX ... [WAIT n|NOWAIT]
> DROP TABLE tbl_name [WAIT n|NOWAIT] ...
> LOCK TABLE ... [WAIT n|NOWAIT]
> OPTIMIZE TABLE tbl_name [WAIT n|NOWAIT]
> RENAME TABLE tbl_name [WAIT n|NOWAIT] ...
> SELECT ... FOR UPDATE [WAIT n|NOWAIT]
> SELECT ... LOCK IN SHARE MODE [WAIT n|NOWAIT]
> TRUNCATE TABLE tbl_name [WAIT n|NOWAIT]
> ```

## 评估 Online DDL 操作的性能

Online DDL 操作的性能取决于是否发生了表的重建。在对大表执行 DDL 操作之前，为了避免影响正常业务操作，最好是先评估一下 DDL 语句的性能再选择如何操作。

1. 复制表结构，创建一个新的表
2. 在新创建的表中插入少量数据
3. 在新表上面执行 DDL 操作
4. 检查执行操作后返回的 `rows affected` 是否是 **0**。如果该值非 **0**，则意味着需要拷贝表数据，此时对 DDL 的上线需要慎重考虑，周密计划

比如

- 修改某一列的默认值（快速，不会影响到表数据）

	```bash
	Query OK, 0 rows affected (0.07 sec)
	```

- 添加索引（需要花费一些时间，但是 `0 rows affected` 说明没有发生表拷贝）

	```bash
	Query OK, 0 rows affected (21.42 sec)
	```

- 修改列的数据类型（需要花费很长时间，并且重建表）

	```bash
	Query OK, 1671168 rows affected (1 min 35.54 sec)
	```

由于在执行  Online  DDL 过程中需要记录并发执行的 DML 操作发生的变更，然后在执行完 DDL 操作之后再应用这些变更，因此使用  Online  DDL 操作花费的时间比不使用 Online 模式执行要更长一些。

##  Online  DDL 支持情况

`INSTANT` 算法支持：**MariaDB 10.3.2**+，**MySQL 8.0.12**+。`NOCOPY` 只支持 MariaDB 10.3.2 以上版本，不支持 MySQL，这里就暂且忽略了。

重点关注是否 **重建表** 和 **支持并发 DML**：不需要重建表，支持并发 DML 最佳。

![Online DDL Select Path](https://ssl.aicode.cc/prometheus/20201019175824.png)

### 二级索引

| 操作                                                | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| --------------------------------------------------- | ------- | ------- | ------ | -------- | ------------ |
| 创建或者添加二级索引                                | ❌       | ✅       | ❌      | ✅        | ❌            |
| 删除索引                                            | ❌       | ✅       | ❌      | ✅        | ✅            |
| 重命名索引 （⚠️MySQL 5.7+，MariaDB 10.5.2+）         | ❌       | ✅       | ❌      | ✅        | ✅            |
| 添加 `FULLTEXT` 索引                                | ❌       | ✅ ①     | ❌ ①    | ❌        | ❌            |
| 添加 `SPATIAL` 索引（⚠️MySQL 5.7+，MariaDB 10.2.2+） | ❌       | ✅       | ❌      | ❌        | ❌            |
| 修改索引类型                                        | ✅       | ✅       | ❌      | ✅        | ✅            |

说明：

- ① 第一次添加全文索引字段时需要重建表，之后就不需要了

### 主键

| 操作                         | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| ---------------------------- | ------- | ------- | ------ | -------- | ------------ |
| 添加主键                     | ❌       | ✅ ②     | ✅ ②    | ✅        | ❌            |
| 删除主键                     | ❌       | ❌       | ✅      | ❌        | ❌            |
| 删除一个主键同时添加一个新的 | ❌       | ✅       | ✅      | ✅        | ❌            |

说明：

- 重建聚簇索引总是需要拷贝表数据（InnoDB 是“索引组织表”），所以最好是在创建表的时候就定义好主键
- 如果创建表是没有指定主键，InnoDB 会选择第一个 `NOT NULL` 的 `UNIQUE` 索引作为主键，或者使用系统生成的 KEY
- ② 对聚簇索引来说，使用 `INPLACE` 模式比 `COPY` 模式要高效一些：不会产生 **undo log** 和 **redo log**，二级索引是有序的，所以可以按顺序加载，不需要使用变更缓冲区

### 普通列

| 操作                          | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| ----------------------------- | ------- | ------- | ------ | -------- | ------------ |
| 列添加                        | ✅ ③     | ✅       | ❌ ③    | ✅ ③      | ❌            |
| 列删除                        | ❌ ④     | ✅       | ✅      | ✅        | ❌            |
| 列重命名                    | ❌       | ✅       | ❌      | ✅ ⑤      | ✅            |
| 改变列的顺序                  | ❌ ⑫     | ✅       | ✅      | ✅        | ❌            |
| 设置默认值                    | ✅       | ✅       | ❌      | ✅        | ✅            |
| 修改数据类型                  | ❌       | ❌       | ✅      | ❌        | ❌            |
| 扩展 `VARCHAR` 长度（⚠️MySQL 5.7+, MariaDB 10.2.2+） | ❌ ⑬     | ✅       | ❌ ⑥    | ✅        | ✅            |
| 删除列的默认值                | ✅       | ✅       | ❌      | ✅        | ✅            |
| 改变自增值                    | ❌       | ✅       | ❌      | ✅        | ❌ ⑦         |
| 设置列为 NULL                 | ❌       | ✅       | ✅ ⑧     | ✅        | ❌            |
| 设置列为 NOT NULL             | ❌       | ✅ ⑨      | ✅ ⑨     | ✅        | ❌            |
| 修改 `ENUM` 和 `SET` 列的定义 | ✅       | ✅       | ❌ ⑩     | ✅        | ✅            |

说明：

- ③ 并发 DML：当插入一个自增列时，不支持并发的 DML 操作，添加自增列时，大量的数据会被重新组织，代价高昂

- ③ 重建表：添加列时，MySQL 5.7及之前版本需要重建表，MySQL 8.0 当 `ALGORITHM=INPLACE` 时，需要重建表，`ALGORITHM=INSTANT` 时不需要重建  

- ③ INSTANT算法：添加列时，使用 `INSTANT` 算法有下面这些限制

  - 添加列操作不能和其它不支持 `INSTANT` 算法的操作合并为一条 `ALTER TABLE` 语句
  - 新增的列只能添加到表的最后，不能放到其它列的前面，在 MariaDB 10.4 之后，支持在任意位置添加
  - 不能将列添加到 `ROW_FORMAT=COMPRESSED` 的表中
  - 不能将列添加到包含 `FULLTEXT` 的表中
  - 不能将列添加到临时表中，临时表只支持 `ALGORITHM=COPY`
  - 不能将列添加到驻留在数据字典表空间中的表中
  - 在添加列的时候不会计算行的大小限制，该限制在执行 DML 操作插入或者更新表时才会被检查

- ④ 删除列时，大量的数据需要被重新组织，代价高昂，在 MariaDB 10.4 之后，删除列支持 INSTANT 算法

- ⑤ 重命名列时，确保只改变列名，不改变数据类型，这样才能支持并发的 DML 操作

- ⑥ 扩展 VARCHAR 长度时，INPLACE 是有条件的，必须保证用于标识字符串长度的长度字节不变（这里说的都是字节，不是 VARCHAR 的字符长度，字节占用与采用的字符集有关，`utf8` 字符集下，一个字符占 3 个字节， `utf8mb4` 则 4 个字节）

  - 当 VARCHAR 列长度在 0-255 个字节时，长度标识占用一个字节
  - 当 VARCHAR 列长度大于 255 个字节时，长度标识占用两个字节

  因此，INPLACE 只支持 0-255 个字节之间或者 256 个字节到更大的长度之间的变更。VARCHAR 列长度减小是不支持 INPLACE 的。

- ⑦ 自增列值变更是修改的内存中的值，不是数据文件

- ⑧ ⑨ 设置列为 `[NOT] NULL` 时，大量的数据被重新组织，代价高昂

- ⑩ 修改 `ENUM` 和 `SET` 类型的列定义时，是否需要表拷贝取决于已有元素的个数和插入成员的位置

- ⑫ 在 MariaDB 10.4 之后，列排序支持 INSTANT 算法

- ⑬ 在 MariaDB 10.4.3  之后，InnoDB 支持使用 INSTANT 算法增加列的长度，但是也有一些限制，具体参考 [Changing the Data Type of a Column](https://mariadb.com/kb/en/innodb-online-ddl-operations-with-the-instant-alter-algorithm/#changing-the-data-type-of-a-column)

### 生成列

| 操作                    | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| ----------------------- | ------- | ------- | ------ | -------- | ------------ |
| 添加 `STORED` 列        | ❌       | ❌       | ✅      | ❌        | ❌            |
| 修改 `STORED` 列的排序  | ❌       | ❌       | ✅      | ❌        | ❌            |
| 删除 `STORED` 列        | ❌       | ✅       | ✅      | ✅        | ❌            |
| 添加 `VIRTUAL` 列       | ✅       | ✅       | ❌      | ✅        | ✅            |
| 修改 `VIRTUAL` 列的排序 | ✅       | ❌       | ✅      | ❌        | ❌            |
| 删除 `VIRTUAL` 列       | ✅       | ✅       | ❌      | ✅        | ✅            |

### 外键

| 操作         | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| ------------ | ------- | ------- | ------ | -------- | ------------ |
| 添加外键约束 | ❌       | ✅ ⑭      | ❌      | ✅        | ✅            |
| 删除外键约束 | ❌       | ✅       | ❌      | ✅        | ✅            |

说明：

- ⑭ 添加外键时，只有当 [`foreign_key_checks`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_foreign_key_checks) 选项被禁用的时候才支持 `INPLACE` 算法

### 表

| 操作                    | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| ----------------------- | ------- | ------- | ------ | -------- | ------------ |
| 修改 `ROW_FORMAT`       | ❌       | ✅       | ✅      | ✅        | ❌            |
| 修改 `KEY_BLOCK_SIZE`   | ❌       | ✅       | ✅      | ✅        | ❌            |
| 设置持久表统计信息      | ❌       | ✅       | ❌      | ✅        | ✅            |
| 指定字符集              | ❌       | ✅       | ✅ ⑮     | ❌        | ❌            |
| 转换字符集              | ❌       | ❌       | ✅ ⑯     | ❌        | ❌            |
| 优化表                  | ❌       | ✅ ⑰      | ✅      | ✅        | ❌            |
| 使用 `FORCE` 选项重建表 | ❌       | ✅ ⑱      | ✅      | ✅        | ❌            |
| 执行空的重建            | ❌       | ✅ ⑲      | ✅      | ✅        | ❌            |
| 重命名表                | ✅       | ✅       | ❌      | ✅        | ✅            |

说明：

- ⑮⑯ 当字符集不同时，需要重建表
- ⑰⑱⑲ 如果表中包含 `FULLTEXT` 的字段，则不支持 INPLACE

### 表空间

| 操作                                     | INSTANT | INPLACE | 重建表 | 并发 DML | 只修改元数据 |
| ---------------------------------------- | ------- | ------- | ------ | -------- | ------------ |
| 重命名常规表空间                         | ❌       | ✅       | ❌      | ✅        | ✅            |
| 启用或者禁用常规表空间加密               | ❌       | ✅       | ❌      | ✅        | ❌            |
| 启用或者禁用 `file-per-table` 表空间加密 | ❌       | ❌       | ✅      | ❌        | ❌            |

### 限制

- 在临时表 `TEMPORARY TABLE` 上创建索引时会发生表拷贝
- 如果表上有 `ON...CASCADE` 或者 `ON...SET NULL` 约束，则 `ALERT TABLE` 不支持字句 `LOCK=NONE`
- 在 Onlne DDL 操作完成之前，它必须等待相关表已经持有元数据锁的事务提交或者回滚，在这个过程中，相关表的新事务会被阻塞，无法执行
- 当在大表上执行涉及到表重建的 DDL 时，会存在以下限制
  - 没有任何机制可以暂停 Online DDL操作或限制 Online DDL 操作的 I/O 或CPU使用率
  - 如果操作失败，则回滚 Online DDL操作的代价非常高昂
  - 长时间运行的 Online  DDL 可能会导致复制延迟。 Online  DDL 操作必须在 Master 上执行完成后才能在 Slave 上执行，在这个过程中， 并发处理的 DML 在 Slave 上面必须等待 DDL 操作完成后才会执行。

## 写在最后

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 参考

- [MariaDB Knowledge Base: InnoDB Online DDL](https://mariadb.com/kb/en/innodb-online-ddl/)
- [MariaDB Knowledge Base: WAIT and NOWAIT](https://mariadb.com/kb/en/wait-and-nowait/)
- [MySQL 5.6:  InnoDB and Online DDL](https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl.html)
- [MySQL 5.7:  InnoDB and Online DDL](https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl.html)
- [MySQL 8.0:  InnoDB and Online DDL](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl.html)
- [极客时间：MySQL实战 45 讲](https://time.geekbang.org/column/intro/139)
