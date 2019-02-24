# 《Unix系统编程》读书笔记-文件I/O缓冲

![](https://ssl.aicode.cc/15510249112782.jpg)


[TOC]

I/O 系统调用会直接将数据传递到内核缓冲区高速缓存，而 stdio 库函数会等到用户空间的流缓冲区填满，再调用 write()将其传递到内核缓冲区高速缓存。
 
![-w791](https://ssl.aicode.cc/15510236846870.jpg)

## 文件I/O的内核缓冲

read()和 write()系统调用在操作磁盘文件时不会直接发起磁盘访问， 而是仅仅在用户空间 缓冲区与内核缓冲区高速缓存（kernel buffer cache）之间复制数据。

执行write系统调用之后，在后续某个时刻，内核会将其缓冲区中的数据写入（刷新至）磁盘。如果在此期间，另一进程试图读取该文件的这几 个字节，那么内核将自动从缓冲区高速缓存中提供这些数据，而不是从文件中。

对输入而言，内核从磁盘中读取数据并存储到内核缓冲区中。 read()调用将从 该缓冲区中读取数据，直至把缓冲区中的数据取完，这时，内核会将文件的下一段内容读入缓冲区高速缓存。

Linux 内核对缓冲区高速缓存的大小没有固定上限。内核会分配尽可能多的缓冲区高速缓 存页，而仅受限于两个因素：可用的物理内存总量，以及出于其他目的对物理内存的需求。

缓冲区如果太小，写入同样大小的文件，需要执行系统调用的次数越多，因此性能会变差。

## stdio标准库的缓冲

当操作磁盘文件时，缓冲大块数据以减少系统调用， C 语言函数库的 I/O 函数（比如， fprintf()、fscanf()、fgets()、fputs()、fputc()、fgetc()）正是这么做的。

### 设置一个 stdio 流的缓冲模式

#### setvbuf()

调用 setvbuf()函数， 可以控制 stdio 库使用缓冲的形式

![-w799](https://ssl.aicode.cc/15510238983471.jpg)

> 打开流后，必须在调用任何其他 stdio 函数 之前先调用 setvbuf()。setvbuf()调用将影响后续在指定流上进行的所有 stdio 操作。

参数mode执行了缓冲类型

- **_IONBF** 不对IO进行缓冲。stderr 默认属于这一类型，从而保证错误能立即输出。
- **_IOLBF** 采用行缓冲 I/O。 指代终端设备的流默认属于这一类型。对于输出流，在输出一个换行符 （除非缓冲区已经填满）前将缓冲数据。对于输入流，每次读取一行数据。
- **_IOFBF** 采用全缓冲 I/O。单次读、写数据（通过read/write系统调用）的大小与缓冲区大小相同。指代磁盘的流默认采用此模式。

#### setbuf()

setbuf()函数构建于 setvbuf()之上， 执行了类似任务

![-w387](https://ssl.aicode.cc/15510241352541.jpg)

setbuf(fp,buf)调用除了不返回函数结果外， 就相当于 setvbuf(fp, buf, (buf != NULL) ? _IOFBF : _IONBF, BUFSIZ)。

#### setbuffer()

setbuffer()函数类似于 setbuf()函数， 但允许调用者指定 buf 缓冲区大小

![-w515](https://ssl.aicode.cc/15510241727002.jpg)

对setbuffer的调用，相当于 setvbuf(fp, buf, (buf != NULL) ? _IOFBUF : _IONBUF, size)。

### 刷新stdio缓冲区

使用 fflush()库函数强制将 stdio 输出 流中的数据（即通过 write()）刷新到内核缓冲区中

![-w801](https://ssl.aicode.cc/15510242199634.jpg)

> fflush()函数应用于输入流， 这将丢弃业已缓冲的输入数据。

若参数 stream 为 NULL， 则 fflush()将刷新所有的 stdio 缓冲区，当关闭相应流时，将自动刷新其 stdio 缓冲区。

若 stdin 和 stdout 指向一终端，那么无论何时从 stdin 中读取输入时，都将隐含调用一次 fflush(stdout)函数。

## 控制文件 I/O 的内核缓冲

### 同步 I/O 完整性

- 同步 I/O 数据完整性（synchronized I/O data integrity completion） ，旨在 确保针对文件的一次更新传递了足够的信息（到磁盘），以便于之后对数据的获取。
- 同步 I/O 文件完整性（Synchronized I/O file integrity completion）是上述 synchronized I/O data integrity completion 的超集。该 I/O 完成模式的区别在于在对文件的一次 更新过程中，要将所有发生更新的文件元数据都传递到磁盘上，即使有些在后续对文件数据 的读操作中并不需要。

### 用于控制文件 I/O 内核缓冲的系统调用

#### fsync

fsync()系统调用将使缓冲数据和与打开文件描述符 fd 相关的所有元数据都刷新到磁盘上

![-w802](https://ssl.aicode.cc/15510243943098.jpg)

> 仅在对磁盘设备（或者至少是其高速缓存）的传递完成后， fsync()调用才会返回

#### fdatasync

fdatasync()系统调用的运作类似于 fsync()， 只是强制文件处于 synchronized I/O data integrity completion 的状态

![-w802](https://ssl.aicode.cc/15510244121286.jpg)

> fdatasync()可能会减少对磁盘操作的次数， 由 fsync()调用请求的两次变为一次

#### sync

sync()系统调用会使包含更新文件信息的所有内核缓冲区（即数据块、指针块、 元数据等） 刷新到磁盘上

![-w247](https://ssl.aicode.cc/15510244274464.jpg)


#### 调用 open() 函数时如指定 O_SYNC 标志

调用 open()函数时如指定 O_SYNC 标志，则会使所有后续输出同步（synchronous）

![-w389](https://ssl.aicode.cc/15510244646108.jpg)

- **O_SYNC** 会使所有后续输出同步（synchronous）
    - 采用 O_SYNC 标志（或者频繁调用 fsync()、fdatasync()或 sync()）对性能的影响极大。
    - 如果需要强制刷新内核缓冲区，那么在设计应用程序时就应考虑是否可以使用大 尺寸的 write()缓冲区， 或者在调用 fsync()或 fdatasync()时谨慎行事，而不是在打开文件时就使用 O_SYNC 标志。
    
- **O_DSYNC** 
    - O_DSYNC 标志要求写操作按照 synchronized I/O data integrity completion 来执行（类似于 fdatasync()）。
    - 与之相映成趣的是 O_SYNC 标志，遵从 synchronized I/O file integrity completion （类似于 fsync()函数）。
    
- **O_RSYNC** 与 O_SYNC 标志或 O_DSYNC 标志配合一起使用，将这些标志对写操作的作用结合到读操作中
    - O_RSYNC 和 O_DSYNC 遵照 synchronized I/O data integrity completion 的要求来完成所有后续读操作。
    - O_RSYNC 和 O_SYNC 遵照 synchronized I/O file integrity completion 的要求来完成所有后续读操作。

## 就 I/O 模式向内核提出建议

posix_fadvise()系统调用允许进程就自身访问文件数据时可能采取的模式通知内核

![-w803](https://ssl.aicode.cc/15510246827668.jpg)

内核可以（但不必非要）根据 posix_fadvise()所提供的信息来优化对缓冲区高速缓存的使 用，进而提高进程和整个系统的性能。调用 posix_fadvise()对程序语义并无影响。

## 绕过缓冲区高速缓存：直接 I/O

始于内核 2.4，Linux 允许应用程序在执行磁盘 I/O 时绕过缓冲区高速缓存，从用户空间直 接将数据传递到文件或磁盘设备。有时也称此为直接 I/O（direct I/O）或者裸 I/O(raw I/O)。

在执行open系统调用时指定O_DIRECT标识，可针对一个单独文件或块设备（比如，一块磁盘）执行直接 I/O。

因为直接 I/O（针对磁盘设备和文件）涉及对磁盘的直接访问， 所以在执行 I/O 时，必须遵守一些限制

- 用于传递数据的缓冲区，其内存边界必须对齐为块大小的整数倍
- 数据传输的开始点，亦即文件和设备的偏移量，必须是块大小的整数倍
- 待传递数据的长度必须是块大小的整数倍

## 混合使用库函数和系统调用进行文件 I/O

![-w806](https://ssl.aicode.cc/15510247704949.jpg)

- 给定一个（文件）流， fileno()函数将返回相应的文件描述符
- fdopen()函数与 fileno()函数的功能相反。 给定一个文件描述符，该函数将创建了一个使用 该描述符进行文件 I/O 的相应流
