# Linux必知必会-理解内存使用统计命令free

![DSC07274](https://oayrssjpa.qnssl.com/DSC07274.jpg)

[TOC]

本文详细介绍了Linux系统中的`free`命令的使用方法以及关键参数的含义，这可能是你见过的关于`free`命令最详细的一篇文章了，绝对值得你收藏。

**free**命令显示了Linux系统中物理内存、交换分区的使用统计信息。

## 指标说明

![](https://oayrssjpa.qnssl.com/14965636958956.jpg)

使用`free`命令查看内存信息，最重要的是理解当前系统的可用内存并不是直接看 **free** 字段就可以看出来的，应该参考的是

    可用内存 = free + buffers + cached

除去标题行之后，第一行为 **物理内存使用统计**：

| 标题 | 说明 |
| --- | --- |
| total | 物理内存总量 **total = used + free** |
| used | 已使用内存总量，包含应用使用量+buffer+cached |
| free | 空闲内存总量 |
| shared | 共享内存总量 |
| buffers | 块设备所占用的缓存 |
| cached | 普通文件数据所占用的缓存 |
| available | 当前可用内存总量（可用于分配给应用的，不包含虚拟内存） |

> 对于`available`字段，在内核3.14中，它会从`/proc/meminfo`中的**MemAvailable**读取，在内核2.6.27+的系统上采用模拟的方式获取，其它情况下直接与**free**的值相同。

第二行**-/+ buffers/cache** 中只有两列**used**和**free**有值，它们是物理内存的调整值

| 标题 | 说明 |
| --- | --- |
| used | 已使用内存（used）减去buffer和cached之后的内存，也就是**应用正在使用的内存总量** |
| free | 空闲内存加上buffer和cached之后的内存，也就是**真正的可用内存总量** |

第三行为交换分区使用统计

| 标题 | 说明 |
| --- | --- |
| total | 交换分区内存总量 |
| used | 正在使用的交换分区内存 |
| free | 空闲交换分区内存 |

在上面这些指标中，我们需要注意的是在下面这些情况下，系统是正常的，不需要担心

- 空闲内存**free**接近于0
- 已使用内存**used**接近于**total**
- 可用内存（**free+buffers/cache**）占**total**的 20% 以上
- 交换分区内存 **swap** 没有发生改变

下面情况说明内存过低，需要注意！

- 可用内存（**free+buffers/cache**）过低，接近于0的时候
- 交换分区内存占用**swap used**增加或者有波动
- `dmesg | grep oom-killer`显示有**OutOfMemory-killer**正在运行

## 常用参数


| 选项 | 说明 |
| --- | --- |
| -b/k/m/g | 以bytes/kilobytes/megabytes/gigabytes为单位显示结果  |
| -h | 以人类可读的方式输出统计结果 |
| -t | 使用该选项会多显示一行标题为Total的统计信息 |
| -o | 禁止显示第二行的缓冲区调整值 |
| -s | 每隔多少秒自动刷新结果 |
| -c | 与**-s**配合使用，控制刷新结果次数 |
| -l | 显示高低内存的统计详情 |
| -a | 显示可用内存 |
| -V | 显示版本号 |

> 版本不同，可能部分选项也不相同。

## 参考示例

    # free -t -a -g

![](https://oayrssjpa.qnssl.com/14965907444264.jpg)


本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。


## 参考文献

- [Meaning of the buffers/cache line in the output of free](https://serverfault.com/questions/85470/meaning-of-the-buffers-cache-line-in-the-output-of-free)
- [Linux ate my ram!](http://www.linuxatemyram.com/)

