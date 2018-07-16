# Linux服务器下的HTTP抓包分析

说到抓包分析，最简单的办法莫过于在客户端直接安装一个Wireshark或者Fiddler了，但是有时候由于客户端开发人员（可能是第三方）知识欠缺或者其它一些原因，无法顺利的在客户端进行抓包分析，这种情况下怎么办呢？

本文中，我们将给大家介绍在服务端进行抓包分析的方法，使用tcpdump抓包，配合Wireshark对HTTP请求进行分析，非常简单有效。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 使用tcpdump在服务器抓包

在服务端进行抓包分析，使用tcpdump

    tcpdump -tttt -s0 -X -vv tcp port 8080 -w captcha.cap

这里的参数是这样的

- **-tttt** 输出最大程度可读的时间戳
- **-s0** 指定每一个包捕获的长度，单位是byte，使用-s0可以捕获整个包的内容
- **-X** 以hex和ASCII两种形式显示包的内容
- **-vv** 显示更加多的包信息
- **tcp** 指我们只捕获tcp流量
- **port 8080** 指我们只捕获端口8080的流量
- **-w captcha.cap** 指定捕获的流量结果输出到captcha.cap文件，便于分析使用

> 关于tcpdump更加高级的用法，可以参考 [tcpdump简明教程][tcpdump-tutorial]

上述命令会保持运行，并将结果输出到 **captcha.cap** 文件中，在这个过程中，所有访问 8080 端口的 TCP 流量都会被捕获。当请求结束之后，我们可以使用 `Ctrl+C` 中断该命令的执行，这时候在当前目录下就可以看到生成了一个名为 **captcha.cap** 的文件。

## 使用Wireshark分析

接下来我们从服务器上下载这个captcha.cap文件到自己电脑上，使用 [Wireshark][wireshark] 打开

> 最简单的下载方法当然是使用scp了
> 
>     scp account@ip:/path/to/captcha.cap .

![](https://oayrssjpa.qnssl.com/15317212624721.jpg)

因为我们需要分析http包，直接打开看显然无法区分我们需要的内容，因此，可以在filter栏中添加过滤规则 `http`，这样就可以只展示http流量了

![](https://oayrssjpa.qnssl.com/15317213481535.jpg)

当请求比较多的时候，我们还是无法快速区分出哪个是指定客户端的访问请求，好在强大的filter可以组合使用

    http and ip.src == 192.168.0.65    

上面这个filter将会过滤出所有来自客户端 192.168.0.65 的http流量。

![](https://oayrssjpa.qnssl.com/15317215272868.jpg)

找到我们需要分析的http请求了，那么怎么查看请求响应的内容呢？也很简单，只需要选中这个请求，右键 **Follow** - **HTTP Stream**：

![](https://oayrssjpa.qnssl.com/15317216039417.jpg)

在新开的窗口中，我们就可以看到这个请求的所有内容了

![](https://oayrssjpa.qnssl.com/15317217869717.jpg)


## 总结

tcpdump和wireshark都是非常强大的网络分析工具，其使用用途不仅仅局限于http请求抓包，借助这两个工具，我们可以对所有的网络流量，网络协议进行分析。本文只是针对最常见的http请求抓包方法做了一个简单的讲解，实际上配合wireshark强大的filter规则，我们可以更加精准的对流量进行过滤，分析。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

[wireshark]: https://www.wireshark.org/#download
[tcpdump-tutorial]: https://github.com/mylxsw/growing-up/blob/master/doc/tcpdump%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B.md
