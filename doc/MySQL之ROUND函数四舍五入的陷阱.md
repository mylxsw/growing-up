# MySQL之ROUND函数四舍五入的陷阱

![FullSizeRender 2](https://oayrssjpa.qnssl.com/2017-01-07-FullSizeRender 2.jpg)

[TOC]

在MySQL中，`ROUND`函数用于对查询结果进行四舍五入，不过最近使用ROUND函数四舍五入时意外发现并没有预期的那样，本文将这一问题记录下来，以免大家跟我一样犯同样的错误。

## 问题描述

假如我们有如下一个数据表`test`，建表语句如下

    CREATE TABLE test (
      id int(11) NOT NULL AUTO_INCREMENT,
      field1 bigint(10) DEFAULT NULL,
      field2 decimal(10,0) DEFAULT NULL,
      field3 int(10) DEFAULT NULL,
      field4 float(15,4) DEFAULT NULL,
      field5 float(15,4) DEFAULT NULL,
      field6 float(15,4) DEFAULT NULL,
      PRIMARY KEY (id)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

我们创建了一个名为`test`的表，出了`id`字段之外还包含了多个字段，拥有这不同的数据类型。我们向这个表中插入一条数据
    
    INSERT INTO test (field1, field2, field3, field4, field5, field6) VALUE (100, 100, 100, 1.005, 3.5, 2.5);
    
插入之后表中的数据是这样的

    mysql> select * from test;
    +----+--------+--------+--------+--------+--------+--------+
    | id | field1 | field2 | field3 | field4 | field5 | field6 |
    +----+--------+--------+--------+--------+--------+--------+
    |  1 |    100 |    100 |    100 | 1.0050 | 3.5000 | 2.5000 |
    +----+--------+--------+--------+--------+--------+--------+
    1 row in set (0.00 sec)

如果现在我们执行下面这个SQL，你觉得结果会是什么样的呢？

    SELECT
      round(field1 * field4),
      round(field2 * field4),
      round(field3 * field4),
      round(field1 * 1.005),
      round(field2 * 1.005),
      round(field3 * 1.005),
      round(field5),
      round(field6)
    FROM test;

最初一直以为这样的结果肯定是都是**101**，因为上面这六个取值结果都是对`100 * 1.005`进行四舍五入，结果肯定都是`101`才对，而后面两个肯定是**4**和**3**才对，但是最终的结果却是与设想的大相径庭

    *************************** 1. row ***************************
    round(field1 * field4): 100
    round(field2 * field4): 100
    round(field3 * field4): 100
     round(field1 * 1.005): 101
     round(field2 * 1.005): 101
     round(field3 * 1.005): 101
             round(field5): 4
             round(field6): 2
    1 row in set (0.00 sec)

## 为什么会这样？

同样是100*1.005，为什么从数据库中的字段相乘得到的结果和直接字段与小数相乘得到的不一样呢？

对这个问题百思不得其解，各种百度谷歌无果。。。没办法，还得靠自己，这个时候最有用的就是官网文档了，于是查询了mysql官方文档中关于ROUND函数的部分，其中包含下面两条规则

- For exact-value numbers, ROUND() uses the “round half up” rule（对于精确的数值，`ROUND`函数使用四舍五入）
- For approximate-value numbers, the result depends on the C library. On many systems, this means that ROUND() uses the “round to nearest even” rule: A value with any fractional part is rounded to the nearest even integer. （对于近似值，则依赖于底层的C函数库，在很多系统中`ROUND`函数会使用“取最近的偶数”的规则）

通过这两条规则，我们可以看出，由于我们在使用两个字段相乘的时候，最终的结果是按照`float`类型处理的，而在计算机中`float`类型不是精确的数，因此处理结果会按照第二条来，而直接整数字段与1.005这样的小数运算的结果是因为两个参与运算的值都是精确数，因此按照第一条规则计算。从`field5`和`field6`执行`ROUND`函数的结果可以明确的看确实是转换为了最近的偶数。

## 总结

从这个例子中可以看到，在MySQL中使用ROUND还是要非常需要注意的，特别是当参与计算的字段中包含浮点数的时候，这个时候计算结果是不准确的。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，另外，求[follow](https://github.com/mylxsw)😂。


