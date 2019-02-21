# 《Unix系统编程》读书笔记-进程凭据

[TOC]

## 概念

在Linux系统中，每个进程都有一套用数字表示的用户ID（UID）和组ID(GID)。有时，也将这些 ID 称之为 **进程凭证**。操作系统内核通过进程凭证来判断当前进程是否可以做执行某个系统调用。

- **实际用户/组ID** 是指登录用户/组的ID，谁启动了进程，那么该进程的实际用户/组ID就是谁的ID。
- **有效用户/组ID** 是指进程用来决定其对资源的访问权限所采用的用户/组ID。一般情况下，有效用户ID就是实际用户ID，当设置了set-user-id/set-group-id权限位时，有效用户/组ID将会被设置为可执行文件的属主用户/组ID。

    - 当进程尝试执行各种操作（系统调用）时，将结合有效用户ID、有效组ID，连同辅助组ID一起来确定授予进程的权限。
    - 有效用户ID为0的进程拥有超级用户的所有权限，这样的进程叫特权级进程。而某些系统调用只能由特权级进程执行。
    
- **set-user-id/set-group-id** 当进程设置了set-user-id/set-group-id权限位时，进程的有效用户/组ID会被设置为可执行文件的属主用户/组ID，这样进程就获得了常规情况下不具备的权限。

    - set-user-id程序会将进程的有效用户ID置为可执行文件的用户ID（属主），从而获得常规情况下并不具有的权限。
    - 非特权用户能够对其拥有的文件进行设置，而特权级用户（CAP_FOWNER）能够对任何文件进行设置。
    - 当运行set-user-ID程序时，内核会将进程的有效用户ID设置为可执行文件的用户ID。set-group-ID程序对进程有效ID的操作类似。
    - 也可以利用程序的set-user-id/set-group-id机制将进程的有效ID修改为非root用户。
    
- **saved set-user-id/set-group-id** 设计保存 set-user-ID（saved set-user-ID）和保存 set-group-ID (saved set-group-ID)， 意在与 set-user-ID 和 set-group-ID 程序结合使用来实现有效用户/组ID在set-user-id/set-group-id和实际用户/组ID之间的切换。**saved set-user-id/set-group-id** 由进程的 **有效用户/组ID** 复制而来。

    执行程序时，依次发生如下事件
    
    1. 若可执行文件的 set-user-ID (set-group-ID)权限位已开启， 则将进程的有效用户（组）ID置为可执行文件的属主。若未设置 set-user-ID (set-group-ID)权限位， 则进程的有效用户（组）ID 将保持不变。
    2. 保存set-user-ID 和保存 set-group-ID 的值由对应的有效 ID 复制而来。无论正在执行的文件是否设置了 set-user-ID 或 set-group-ID 权限位，这一复制都将进行。

- **文件系统用户ID和组ID** 打开文件，改变文件属主，修改文件权限等文件系统操作，决定其操作权限的是文件系统用户ID和组ID(结合辅助组ID)，而非有效用户ID和组ID。
- **辅助组ID** 用于标识进程所述的若干附加的组。新进程从其父进程继承这些ID，登录shell从系统组文件中获取其辅助组ID。

## 设置set-user-id权限位

在Linux系统中，使用 `chmod +s` 为文件设置 set-user-id 权限位。

    chmod +s filename

比如我们创建如下的Go语言程序来做个演示

    package main
    
    import (
    	"fmt"
    	"syscall"
    )
    
    func main() {
    	fmt.Printf("uid=%d, euid=%d\n", syscall.Getuid(), syscall.Geteuid())
    }

该程序执行时会输出当前的**实际用户ID**和**有效用户ID**

编译该程序

    $ go build main.go 
    $ ll
    total 3968
    -rwxr-xr-x  1 mylxsw  staff   1.9M Feb 22 00:22 main
    -rw-r--r--  1 mylxsw  staff   130B Feb 22 00:20 main.go

修改该程序的属主

    $ sudo chown root main
    $ ll
    total 3968
    -rwxr-xr-x  1 root    staff   1.9M Feb 22 00:22 main
    -rw-r--r--  1 mylxsw  staff   130B Feb 22 00:20 main.go

执行main程序

    $ ./main
    uid=501, euid=501

设置main文件的set-user-id权限位

    $ sudo chmod +s main
    $ ll
    total 3968
    -rwsr-sr-x  1 root    staff   1.9M Feb 22 00:22 main
    -rw-r--r--  1 mylxsw  staff   130B Feb 22 00:20 main.go

再次执行main程序

    $ ./main
    uid=501, euid=0

我们可以看到，进程的有效用户ID变成了0，也就是特权用户。虽然我们是以普通用户的身份来执行的main程序，但是在main程序执行时，实际上是有用root权限的，可以访问root用户的资源。

## 相关系统调用

![-w471](https://ssl.aicode.cc/15507653167807.jpg)


### 获取实际ID

![-w638](https://ssl.aicode.cc/15507654381270.jpg)

### 获取有效ID

![-w641](https://ssl.aicode.cc/15507654771879.jpg)


### 修改有效ID

![-w632](https://ssl.aicode.cc/15507655058044.jpg)

当非特权进程调用 setuid()时， 仅能修改进程的有效用户 ID。 而且，仅能将有效用户 ID 修改成相应的实际用户 ID 或保存 set-user-ID。
	
当特权进程以一个非 0 参数调用 setuid()时， 其实际用户 ID、 有效用户 ID 和保存 set-user-ID 均被置为 uid 参数所指定的值。这一操作是单向的，一旦特权进程以此方式修改了其 ID， 那么所有特权都将丢失，且之后也不能再使用 setuid()调用将有效用户 ID 重置为 0。
	
由于对组 ID 的修改不会引起 进程特权的丢失（拥有特权与否由有效用户 ID 决定），特权级程序可以使用 setgid()对组 ID 进行任意修改。

![-w636](https://ssl.aicode.cc/15507655355395.jpg)


非特权级进程仅能将其有效 ID 修改为相应的实际 ID 或者保存设置 ID。
	
特权级进程能够将其有效 ID 修改为任意值。若特权进程使用 seteuid()将其有效用户 ID 修 改为非 0 值，那么此进程将不再具有特权

### 修改实际ID和有效ID

![-w631](https://ssl.aicode.cc/15507655588791.jpg)


### 同时获取实际、有效和保存设置ID

![-w635](https://ssl.aicode.cc/15507655746006.jpg)


### 同时修改实际、有效和保存设置ID

![-w641](https://ssl.aicode.cc/15507655872786.jpg)


### 获取和修改文件系统ID

![-w634](https://ssl.aicode.cc/15507656009895.jpg)

所有修改进程有效用户 ID 或组 ID 的系统调用总是会修改相应的文件系统 ID。
	
必须使用 Linux 特有的系统调用setfsuid/setfsgid才能，独立于有效 ID 而修改文件系统 ID。

### 获取和修改辅助组ID

![-w636](https://ssl.aicode.cc/15507656190983.jpg)

![-w633](https://ssl.aicode.cc/15507656357141.jpg)

### 修改进程凭证的接口一览表

![-w746](https://ssl.aicode.cc/15507627298801.jpg)
