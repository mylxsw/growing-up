# Linux命令必知必会

[TOC]

## top命令
监控系统的运行状态，并且可以按照cpu、内存、执行时间进行排序。

![top](https://ssl.aicode.cc/2016-10-31-top.png)


第一行中，`03:30:22`是当前时间，`up 39 min`是系统运行的运行了多长时间，`1 user`指出了当前有几个用户登录到系统，`load average`指的是系统负载，这后面的三个值分别是1分钟，5分钟，15分钟的系统负载平均值。

> 如果仅仅需要第一行中的信息，可以使用`uptime`命令。

第二行中，`Task`指出了当前系统有多少个进程，以及各种状态的进程统计信息。

<!--more-->

第三行是`%Cpu(s)`，代表了CPU占用比例，其中：

- **us** 用户模式(*user* mode)
- **sy** 系统模式(*system* mode)
- **ni** 优先值(low priority user mode(*nice*))
- **id** 空闲CPU百分比(*idle* task)
- **wa** 等待输入输出的CPU事件百分比(I/O *waiting*)
- **hi** servicing IRQs
- **si** servicing soft IRQs
- **st** *steal* (time given to other DomU instances)

> **ni**是优先值(nice value)，也就是任务的优先值。优先值为负数，则说明任务有更高的优先级，正数值说明任务有更低的优先级，该值为0意味着进程的优先级没有调整。

最后两行为内存信息，前者`Mem`为物理内存占用信息，后者`Swap`为交换分区占用信息。

> 使用`-M`参数可以更加友好的显示内存占用信息。默认是以kb展示的，看起来比较费劲，使用`-M`之后会根据数值大小，以G/M为单位展示。

最下面是进程的信息区域：

- **PID** 进程的PID
- **USER** 用户名，任务属主
- **PR** 任务的优先级
- **NI** 优先值
- **VIRT** 虚拟映像（kb），任务当前使用的虚拟内存数量
- **RES** 常驻物理内存占用量，RES=CODE+DATA
- **SHR** 共享内存大小（kb）
- **S** 进程状态（D-不可中断的睡眠，R-运行，S-睡眠，T-停止，Z-僵尸进程）
- **%CPU** CPU使用量
- **%MEM** 内存使用量
- **TIME+** CPU时间，百分之一
- **COMMAND** 程序名称

> 参考[linux top命令详解](http://blog.csdn.net/sanshiqiduer/article/details/1933625)


## pgrep/pkill 命令

根据名称或者其它属性查询（发送信号）进程信息。

`pgrep`命令根据提供的条件查询进程的pid，查询条件是and方式的，对于同一个选项，使用『,』分隔可以按照or方式查询。


    pgrep -u root sshd   # 查询进程名为sshd，并且属主是root的进程
    pgrep -u root,daemon # 查询属主是root或者daemon的进程


`pkill` 使用与`pgrep`类似，不过它不是用来查询进程pid，而是给进程发送信号，默认会发送 **SIGTERM**信号。

例如:

    $ pgrep -u root named # 查找named进程的pid
    $ pkill -HUP syslogd  # 告诉syslogd重新读取配置文件


> 要查看有哪些信号可用，可以使用`kill -l`列出所有的信号以及其数值。



## except命令

- **send** 发送一个字符串给进程。
- **expect** 等待来自进程返回的字符串。
- **spawn** 开始一个命令。

### 实现控制台SSH直接登陆Linux服务器

    #!/usr/bin/expect

    set timeout 20

    set ip "IP地址"
    set user "用户名"
    set password "密码"

    spawn ssh "$user\@$ip"

    expect "$user@$ip's password:"
    send "$password\r"

    interact

> 参考 [6 Expect Script Examples to Expect the Unexpected (With Hello World)
](http://www.thegeekstuff.com/2010/10/expect-examples/)


## pstack命令

`pstack`是一个shell脚本，用于打印正在运行的进程的栈跟踪信息，它实际上是`gstack`的一个链接。

该命令只需要提供一个参数，进程的pid即可。

    $ sudo pstack $(pgrep -uroot php-fpm)
    [sudo] password for guanyy:
    #0  0x000000380d8e86f3 in __epoll_wait_nocancel () from /lib64/libc.so.6
    #1  0x00000000007ec4a4 in fpm_event_epoll_wait ()
    #2  0x00000000007e1517 in fpm_event_loop ()
    #3  0x00000000007dc887 in fpm_run ()
    #4  0x00000000007e3bd8 in main ()

> `pstack`是gdb的一部分，如果系统没有pstack命令，使用yum搜索安装`gdb`即可。

## strace命令

`strace`命令用于跟踪系统调用和信号。主要用于诊断，调试程序，使用该命令能够打印出进程执行的系统调用信息。

> 在 Mac 下使用`dtruss`命令代替

### 找出应用程序启动时读取的配置文件

    $ strace php 2>&1 | grep php.ini
    open("/usr/local/bin/php.ini", O_RDONLY) = -1 ENOENT (No such file or directory)
    open("/usr/local/lib/php.ini", O_RDONLY) = 4
    lstat64("/usr/local/lib/php.ini", {st_mode=S_IFLNK|0777, st_size=27, ...}) = 0
    readlink("/usr/local/lib/php.ini", "/usr/local/Zend/etc/php.ini", 4096) = 27
    lstat64("/usr/local/Zend/etc/php.ini", {st_mode=S_IFREG|0664, st_size=40971, ...}) = 0


> 这里的`2>&1` 是将标准错误输出重定向到标准输出。

### 查找为什么程序没有打开指定文件

    $ strace -e open,access 2>&1 |grep your-filename


`-e`参数指定了一个限定表达式用于指定要跟踪的事件和如何跟踪它们。

    [qualifier=][!]value1[,value2]...


这里的`qualifier`可选值为: `trace`, `abbrev`, `verbose`, `raw`, `signal`, `read`, `write`。默认的`qualifier`是`trace`。

### 查看进程正在执行什么操作

    root@dev:~# strace -p 15427
    Process 15427 attached - interrupt to quit
    futex(0x402f4900, FUTEX_WAIT, 2, NULL
    Process 15427 detached


`-p`指定了strace跟踪的进程的pid，这样就避免了每次执行strace时需要重启程序。

### 查看进程的哪些操作比较耗时

    root@dev:~# strace -c -p 11084
    Process 11084 attached - interrupt to quit
    Process 11084 detached
    % time     seconds  usecs/call     calls    errors syscall
    ------ ----------- ----------- --------- --------- ----------------
     94.59    0.001014          48        21           select
      2.89    0.000031           1        21           getppid
      2.52    0.000027           1        21           time
    ------ ----------- ----------- --------- --------- ----------------
    100.00    0.001072                    63           total


`-c`参数用于统计进程做了哪些系统调用，调用的时间统计等，并对这些信息做一个汇总显示。

### 查看为什么xxx无法连接到服务器

    $ strace -e poll,select,connect,recvfrom,sendto nc www.news.com 80
    sendto(3, "\\24\\0\\0\\0\\26\\0\\1\\3\\255\\373NH\\0\\0\\0\\0\\0\\0\\0\\0", 20, 0, {sa_family=AF_NETLINK, pid=0, groups=00000000}, 12) = 20
    connect(3, {sa_family=AF_FILE, path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
    connect(3, {sa_family=AF_FILE, path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
    ...



> 参考[5 simple ways to troubleshoot using Strace](http://www.hokstad.com/5-simple-ways-to-troubleshoot-using-strace)


## nc命令

该命令用于创建任意的TCP/UDP连接或者是监听连接。

### 建立一个基本的C/S模型(文件远程复制)

在Server1上，使用nc命令创建一个服务端：

    server1 $ nc -l 1234


在Server2上，使用nc作为客户端连接到server1

    server2 $ nc server1的IP地址 1234


这样就建立起一个简单的C/S连接，在server2中输入任何内容，在server1都可以接受到（同步显示）。

上面的例子可以改造实现文件远程发送

    server1 $ nc -l 1234 > filename.out

在server2上

    server2 $ nc server1的IP地址 1234 < filename.in


> `-l` 指定了nc应该作为server端监听指定的端口

### 模拟HTTP请求

    # echo -n "GET / HTTP/1.0\r\n\r\n" | nc php.net 80
    HTTP/1.1 400 Bad Request
    Server: nginx/1.6.2
    Date: Tue, 16 Dec 2014 08:09:35 GMT
    Content-Type: text/html
    Content-Length: 172
    Connection: close

    <html>
    <head><title>400 Bad Request</title></head>
    <body bgcolor="white">
    <center><h1>400 Bad Request</h1></center>
    <hr><center>nginx/1.6.2</center>
    </body>
    </html>


### 端口扫描

端口扫描的作用还是比较大的，使用`nc`可以方便的进行端口扫描。

    # nc -z letv.com 1-100
    Connection to letv.com 22 port [tcp/ssh] succeeded!
    Connection to letv.com 80 port [tcp/http] succeeded!


这里的`1-100`指定了扫描的端口范围，`-z`参数告诉nc命令只报告开放的端口。

> 默认`nc`命令发送的是tcp请求，通过指定参数`-u`可以发送udp请求。

### 目录传输

下面例子中，将server2的phpredis-master目录拷贝到server1。

server1:

    # nc -l 1234|tar zxvf -

server2:

    # tar zcvf - phpredis-master|nc server1的IP地址 1234


> 参考[Linux nc命令详解](http://blog.csdn.net/wang7dao/article/details/7684998)


## pstree命令

该命令用于显示进程树，以树的形式显示正在运行的进程，树的根节点是指定的pid（忽略则为init进程）。


    [root@cdn ~]# pstree -p $(pgrep -uroot php-fpm)
    php-fpm(5445)─┬─php-fpm(5446)
                  ├─php-fpm(5447)
                  ├─php-fpm(5448)
                  ├─php-fpm(7540)
                  ├─php-fpm(21639)
                  └─php-fpm(24727)




## ss命令

`ss`命令用于显示socket的统计信息。

### 显示socket的汇总信息

`-s`选项用于显示汇总信息。


    # ss -s
    Total: 247 (kernel 290)
    TCP:   214 (estab 68, closed 130, orphaned 0, synrecv 0, timewait 130/0), ports 135

    Transport Total     IP        IPv6
    *	  290       -         -
    RAW	  0         0         0
    UDP	  11        7         4
    TCP	  84        81        3
    INET	  95        88        7
    FRAG	  0         0         0


### 查看所有打开的网络端口

`-l`选项用于列出当前正在监听的socket。


    # ss -l
    State      Recv-Q Send-Q      Local Address:Port          Peer Address:Port
    LISTEN     0      128             127.0.0.1:smux                     *:*
    LISTEN     0      128             127.0.0.1:9000                     *:*
    LISTEN     0      50                      *:3306                     *:*
    LISTEN     0      1024                   :::11211                   :::*


使用`ss -pl`可以查看使用网络端口的进程名称，这里的`-p`选项用于显示进程信息。

    # ss -pl
    State      Recv-Q Send-Q      Local Address:Port          Peer Address:Port
    LISTEN     0      128             127.0.0.1:smux                     *:*        users:(("snmpd",1256,8))
    LISTEN     0      50                      *:3306                     *:*        users:(("mysqld",17651,10))
    LISTEN     0      1024                   :::11211                   :::*        users:(("memcached",1849,34))
    LISTEN     0      1024                    *:11211                    *:*        users:(("memcached",1849,33))
    LISTEN     0      511             127.0.0.1:6379                     *:*        users:(("redis-server",1403,4))


> 使用`ss -pl|grep 端口号`查看端口被那个进程占用。

### 显示所有的TCP/UDP Socket

参数`-a`(`--all`)用于显示所有的socket，`-t`指的是TCP， `-u`是UDP, `-w`是RAW, `-x`是UNIX。

    # ss -t -a
    # ss -u -a
    # ss -w -a
    # ss -x -a




> 参考[ss: Display Linux TCP / UDP Network and Socket Information
](http://www.cyberciti.biz/tips/linux-investigate-sockets-network-connections.html)



## w/who命令

`w`命令用于查看当前哪些用户登录到系统和他们正在做什么，`who`命令仅用于查看哪些用户登录系统。

    # w
     15:39:08 up 126 days, 22:35,  3 users,  load average: 0.02, 0.05, 0.02
    USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/0    10.58.92.228     13:29    1:35m  0.03s  0.03s -bash
    root     pts/1    10.58.93.56      10:32    5:06m  0.00s  0.00s -bash
    root     pts/4    10.58.88.20      12:29    0.00s  0.20s  0.00s w
    # who
    root     pts/0        2014-12-18 13:29 (10.58.92.228)
    root     pts/1        2014-12-18 10:32 (10.58.93.56)
    root     pts/4        2014-12-18 12:29 (10.58.88.20)



## iostat

报告CPU的统计信息，设备、分区、网络文件系统（NFS）的I/O统计信息。


    # iostat
    Linux 2.6.32-903.279.9.1.el6.x86_64 (localhost) 	2014年12月18日 _x86_64_	(2 CPU)

    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.35    0.00    0.34    0.42    0.15   98.74

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    vda               4.01         0.35        56.76    3866731  622586087
    dm-0              3.29         0.09        26.33     989378  288796192
    dm-1              3.45         0.05        27.60     554922  302727584
    dm-2              0.32         0.21         2.83    2296845   31060799


这里对几个性能指标进行解释：

- **tps** 每秒发送的I/O请求数
- **Blk_read/s** 每秒读取的block数
- **Blk_wrtn/s** 每秒写入的block数
- **Blk_read** 读取的block数
- **Blk_wrtn** 写入的block数

> 通过指定`-d`参数可以设定自动按照指定时间间隔显示统计信息。例如，下列命令每隔2s显示一次。

    $ iostat -d 2



## iptraf 命令：实时网络统计

交互式的IP网络实时监控工具，图形化界面，比较方便。

    # iptraf


界面如下:

![iptraf](https://oayrssjpa.qnssl.com/2016-10-31-iptraf.png)

> 参考[20 Linux System Monitoring Tools Every SysAdmin Should Know](http://www.cyberciti.biz/tips/top-linux-monitoring-tools.html)


##  查看Linux的版本（Red Hat/Cent OS）

在RedHat和Cent OS下，使用如下命令查看当前系统的版本。

    $ cat /etc/centos-release
    CentOS release 6.3 (Final)


##  time命令： 统计程序执行时间

用于统计程序执行时间，这些事件包含程序从被调用到终止的时间，用户CPU时间，系统CPU时间。

<!--more-->

    $ time ls
    bakup                PDO-1.0.3.tgz     rinetd.tar.gz     yaf-2.2.9.tgz
    channel.xml          package2.xml      PDO_MYSQL-1.0.2      xhprof-0.9.4      zendopcache-7.0.3
    go-pear.phar         package.xml       PDO_MYSQL-1.0.2.tgz  xhprof-0.9.4.tgz  zendopcache-7.0.3.tgz
    PDO-1.0.3            rinetd            yaf-2.2.9

    real    0m0.002s
    user    0m0.000s
    sys 0m0.001s

##  tee命令

`tee`命令用于将标准输入拷贝到标准输出。

    $ echo "hello,world"|tee -a test.txt

上述命令将hello,world字符串输出到test.txt文件中,**-a** 默认情况下，`tee`命令会使用`>`覆盖输出到文件，使用-a属性，会使用`>>`追加方式

##  netstat命令

查看端口占用情况

    # netstat -apn

- **-a**（--all） 显示所有的socket信息（包括监听和未监听）
- **-p**（--program） 显示每个socket所属于的进程名称和PID
- **-n**（--numeric） 显示数字形式的地址而不是符号化的主机名、端口或者用户名


##  perf命令

`perf`命令是随Linux内核代码一同发布和维护的性能诊断工具，由内核社区负责维护和发展。Perf不仅可以用于应用程序性能统计分析，也可以应用于内核代码的的性能统计和分析。

在Cent OS系统上，如果没有该命令的话，可以使用yum进行安装。

    # yum install perf

`perf`命令非常强大，详细介绍的话篇幅比较长，可以阅读这篇文章 [Perf -- Linux下的系统性能调优工具][]。

    用法: perf [--version] [--help] COMMAND [ARGS]

     最常用的perf命令:
       annotate        读取perf.data (使用perf record创建)文件并且显示标注的代码
       archive         Create archive with object files with build-ids found in perf.data file
       bench           进行基准测试的框架工具集
       buildid-cache   Manage build-id cache.
       buildid-list    List the buildids in a perf.data file
       diff            Read perf.data files and display the differential profile
       evlist          List the event names in a perf.data file
       inject          Filter to augment the events stream with additional information
       kmem            Tool to trace/measure kernel memory(slab) properties
       kvm             Tool to trace/measure kvm guest os
       list            列出所有事件类型的符号
       lock            分析锁事件
       mem             分析对内存的访问
       record          运行一个命令并且记录它的分析结果到perf.data文件中
       report          读取perf.data文件并且显示分析结果
       sched           Tool to trace/measure scheduler properties (latencies)
       script          Read perf.data (created by perf record) and display trace output
       stat            运行一个命令并且收集性能计数统计信息
       test            运行可用性测试
       timechart       Tool to visualize total system behavior during a workload
       top             系统分析工具.
       trace           受strace启发创建的工具
       probe           定义一个新的动态跟踪点

     See 'perf help COMMAND' for more information on a specific command.

### perf stat

`perf stat`通过概括精简的方式提供被调试程序运行的整体情况和汇总数据。

创建如下C程序test.c

    #include <stdio.h>

    int main()
    {
        int i = 1;
        while (1) {
            if (i == 100000) break;
            i ++;
        }
        return 0;
    }

编译`gcc test.c -o test`。

    $ perf stat ./test

     Performance counter stats for './test':

              0.837322 task-clock                #    0.747 CPUs utilized          
                     1 context-switches          #    0.001 M/sec                  
                     0 CPU-migrations            #    0.000 M/sec                  
                    98 page-faults               #    0.117 M/sec                  
               269,259 cycles                    #    0.322 GHz                     [90.39%]
               897,270 stalled-cycles-frontend   #  333.24% frontend cycles idle   
               226,746 stalled-cycles-backend    #   84.21% backend  cycles idle   
               764,602 instructions              #    2.84  insns per cycle        
                                                 #    1.17  stalled cycles per insn
               267,843 branches                  #  319.881 M/sec                  
                 3,467 branch-misses             #    1.29% of all branches         [80.37%]

           0.001121130 seconds time elapsed


第一个`task-clock`是CPU利用率，该值比较高，说明该程序属于CPU密集型。第二个`context-switches`是进程上下文切换次数，频繁的切换次数应该是要避免的。

### perf top

用于实时显示当前系统的性能统计信息。该命令主要用来观察整个系统当前的状态，比如可以通过查看该命令的输出来查看当前系统最耗时的内核函数或某个用户进程。

> 执行该命令需要root权限。

使用方法如下

    $ sudo perf top

程序会与top命令类似，动态输出以下内容

    Samples: 1K of event 'cpu-clock', Event count (approx.): 8071695
     39.60%  [kernel]             [k] __do_softirq
     13.46%  [kernel]             [k] _raw_spin_unlock_irqrestore
      9.37%  [kernel]             [k] VbglGRPerform
      8.47%  [kernel]             [k] e1000_xmit_frame
      6.01%  [kernel]             [k] finish_task_switch
      5.82%  [kernel]             [k] e1000_clean
      5.15%  [kernel]             [k] native_read_tsc
      4.75%  [kernel]             [k] kmem_cache_free
      1.32%  [kernel]             [k] tick_nohz_idle_enter
      1.28%  libc-2.17.so         [.] __strstr_sse2
      1.22%  libc-2.17.so         [.] __memset_sse2
      0.82%  libc-2.17.so         [.] __GI___strcmp_ssse3
      0.42%  libpython2.7.so.1.0  [.] 0x000000000007e7c6
      0.42%  libc-2.17.so         [.] __strchrnul
      0.39%  [kernel]             [k] e1000_alloc_rx_buffers
      0.38%  libz.so.1.2.7        [.] 0x0000000000002d76
      0.24%  [kernel]             [k] tick_nohz_idle_exit
      0.21%  [kernel]             [k] kfree

### perf report/record

使用 top 和 stat 之后，您可能已经大致有数了。要进一步分析，便需要一些粒度更细的信息。比如说您已经断定目标程序计算量较大，也许是因为有些代码写的不够精简。那么面对长长的代码文件，究竟哪几行代码需要进一步修改呢？这便需要使用 perf record 记录单个函数级别的统计信息，并使用 perf report 来显示统计结果。

创建新的C程序test3，代码如下

    #include <stdio.h>

    void test();

    int main()
    {
      test();
      return 0;
    }

    void test()
    {
      long i;
      for (i = 0; i < 10000000; i ++) {

      }
      puts("finished");
    }

编译后，执行如下命令

    $ perf record ./test3
    $ perf report

输出以下内容

    Samples: 68  of event 'cpu-clock', Event count (approx.): 17000000
     97.06%  test3  test3              [.] test
      1.47%  test3  [kernel.kallsyms]  [k] __do_softirq
      1.47%  test3  [kernel.kallsyms]  [k] queue_work_on

从中可以看到，大部分时间都消耗在了test函数中。

> `perf record`命令增加`-g`参数可以记录函数的调用图信息。更多详情参考: [Perf -- Linux下的系统性能调优工具][]

##  lsof命令: 列出打开的文件

工具`lsof`是一个可以列出操作系统打开的文件的工具，在Linux系统中，任何事物都是以文件的形式存在，通过文件不仅可以访问常规文件，还可以访问网络连接和硬件设备。

在终端下直接输入`lsof`命令，会列出当前系统打开的所有文件，因为它需要列出核心内存和各种文件，所以必须使用root用户运行才能显示详细的信息。

    COMMAND     PID      USER   FD      TYPE             DEVICE  SIZE/OFF       NODE NAME
    init          1      root  cwd       DIR              253,0      4096          2 /
    init          1      root  rtd       DIR              253,0      4096          2 /
    init          1      root  txt       REG              253,0    150352      10973 /sbin/init
    init          1      root  mem       REG              253,0     65928     264638 /lib64/libnss_files-2.12.so
    init          1      root  mem       REG              253,0   1922112     265339 /lib64/libc-2.12.so
    init          1      root  mem       REG              253,0     93224     277540 /lib64/libgcc_s-4.4.6-20120305.so.1
    init          1      root  mem       REG              253,0     47064     267086 /lib64/librt-2.12.so
    ...

这里的***COMMAND***是进程名称，***PID,USER***分别指的是进程的ID和进程所有者，***FD***是文件描述符，***TYPE***是文件类型，***DEVICE***是磁盘名称，***SIZE***是文件大小，***NODE***是索引节点（文件在磁盘上的标识），***NAME***是打开文件的确切名称。

对于***FD***的值，*cwd*表示当前工作目录，*Lnn*表示类库引用，*mem*表示内存映射文件，*rtd*表示根目录，*pd*表示父目录，*txt*表示进程的数据和代码。

### 常用参数及说明

- lsof **filename** 显示打开指定文件的所有进程
- lsof **-a** 表示两个参数都必须满足时才显示结果
- lsof **-c string** 显示COMMAND列中包含指定字符的进程所有打开的文件
- lsof **-u username** 显示所属user进程打开的文件
- lsof **-g gid** 显示归属gid的进程情况
- lsof **+d /DIR/** 显示目录下被进程打开的文件
- lsof **+D /DIR/** 同上，但是会搜索目录下的所有目录，时间相对较长
- lsof **-d FD** 显示指定文件描述符的进程
- lsof **-n** 不将IP转换为hostname，缺省是不加上-n参数
- lsof **-i** 用以显示符合条件的进程情况
- lsof **-p PID** 选择指定PID
- lsof **-i[46] [protocol][@hostname|hostaddr][:service|port]**
    46: IPv4 or IPv6
    protocol: TCP or UDP
    hostname: Internet host name
    hostaddr: IPv4地址
    service: /etc/service中的 service name (可以不只一个)
    port: 端口号 (可以不只一个)

> 参考: [百度文库][]

##  unzip命令

unzip命令用于解压`.zip`文件，常用参数如下

- **-f** 只更新磁盘上已经存在的文件
- **-u** 更新磁盘上存在的文件，文件不存在则创建
- **-o** 如果文件已经存在则直接覆盖，不提示
- **-d** 指定解压到的目录

例如，解压`test.zip`到`/var/www`目录，部署web站点

    # unzip -u -o -d /var/www test.zip
    # chown -R www:www /var/www


## 使用pushd和popd命令快速切换目录
经常会有这么一种情况，我们会在不同目录中进行频繁的切换，如果目录很深，那么使用`cd`命令的工作量是不小的，这时可以使用`pushd`和`popd`命令快速切换目录。


    $ pwd
    /Users/mylxsw/codes/php/lecloud/api
    $ pushd .
    ~/codes/php/lecloud/api ~/codes/php/lecloud/api
    $ cd ../album/
    $ pwd
    /Users/mylxsw/codes/php/lecloud/album
    $ popd
    ~/codes/php/lecloud/api
    $ pwd
    /Users/mylxsw/codes/php/lecloud/api

## SCP

在服务器和本地计算机之间传递文件

    usage: scp [-12346BCEpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
               [-l limit] [-o ssh_option] [-P port] [-S program]
               [[user@]host1:]file1 ... [[user@]host2:]file2


从服务器下载文件

    scp username@服务器地址:/path/文件名 本地保存路径

上传文件到服务器

    scp 本地文件路径 username@服务器地址:/保存到服务器的路径

> Tip: 如果要操作的对象是整个目录的话，需要添加`-t`参数。

使用范例:


    localhost:Downloads mylxsw$ scp guanyy@10.10.10.10:/home/guanyy/download.db ./
    guanyy@10.10.10.10's password:
    download.db                                   100%   25MB  24.7MB/s   00:01


## Mac OS 清理DNS缓存

    dscacheutil -flushcache


## Cent OS服务器安装PHP的pecl
想要安装某个PHP扩展，但发现服务器上没有pecl，因此需要安装pecl。

    $ sudo yum install php-pear

安装pear之后，pecl就有了。

## 在当前目录下查找大小超过100MB的文件

    find . -type f -size +100M

## 转换iso格式为dmg

    hdiutil convert -format UDRW -o ubuntu-16.04-desktop-amd64.img ubuntu-16.04-desktop-amd64.iso

## 查看磁盘设备

    diskutil list

> 卸载磁盘： `diskutil umountDisk /dev/disk1`

## 拷贝镜像到U盘

    dd if=yourimage.img of=/dev/sdb1


## 使用基于文本的图形界面配置命令setup
安装setup命令

    $ sudo yum install setuptool


安装之后，需要安装一些常见的系统配置组件，也是使用yum：

    $ sudo yum install system-config-services system-config-firewall system-config-network-tui


更多的配置组件可以使用`yum search system-config`命令查看，进入配置界面直接使用`setup`命令。

## 使用sed批量操作文件

下面这个命令实现了批量把符合`.env*`规则的文件中，删除包含`APP_TIMEZONE`的行，追加两行新的配置

    find . -name '.env*' -exec sed -i '' -e '/APP_TIMEZONE/d' -e '$ a \
    DB_TIMEZONE=+08:00\
    APP_TIME_ZONE=PRC\
    ' {} \;
    
下面的命令实现了批量替换符合`.env*`规则的文件中的`APP_TIME_ZONE`为`APP_TIMEZONE`

    find . -name '.env*' -exec sed -i '' -e 's/APP_TIME_ZONE/APP_TIMEZONE/' {} \;

## 查看路由规则

    [root@tristan]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    192.168.99.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
    127.0.0.0       0.0.0.0         255.0.0.0       U     0      0        0 lo
    0.0.0.0         192.168.99.254  0.0.0.0         UG    0      0        0 eth0
    [root@tristan]# ip route show
    192.168.99.0/24 dev eth0  scope link 
    127.0.0.0/8 dev lo  scope link 
    default via 192.168.99.254 dev eth0


[Perf -- Linux下的系统性能调优工具]:http://www.ibm.com/developerworks/cn/linux/l-cn-perf1/
[百度文库]:http://baike.baidu.com/link?url=VXbFBeisjSpMDZzkUQlNiDZrCAi6p7q1TJcgbCT4J4k4mxcU2fyoYOj1Vz8KCBBAKeTJ5qNeeqTnGYhMAh-zfK
