# 三十分钟学会SED

![](https://oayrssjpa.qnssl.com/2016-11-27-14802608583950.jpg)

本文承接之前写的[三十分钟学会AWK][]一文，在学习完AWK之后，趁热打铁又学习了一下SED，不得不说这两个工具真的堪称文本处理神器，谁用谁知道！本文大部分内容依旧是翻译自Tutorialspoint上的入门教程，这次是 [Sed Tutorial](http://www.tutorialspoint.com/sed/index.htm) 一文，内容做了一些删减和补充，增加了一些原文中没有提及到的语法和命令的讲解，并且对原文所有的示例都一一进行了验证，希望本文对大家学习和了解Sed有所帮助。

<!-- more -->

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star。

[TOC]

## 概述

SED的英文全称是 **Stream EDitor**，它是一个简单而强大的文本解析转换工具，在1973-1974年期间由贝尔实验室的*Lee E. McMahon*开发，今天，它已经运行在所有的主流操作系统上了。

*McMahon*创建了一个通用的行编辑器，最终变成为了SED。SED的很多语法和特性都借鉴了**ed**编辑器。设计之初，它就已经支持正则表达式，SED可以从文件中接受类似于管道的输入，也可以接受来自标准输入流的输入。

SED由自由软件基金组织（FSF）开发和维护并且随着GNU/Linux进行分发，因此，通常它也称作 **GNU SED**。对于新手来说，SED的语法看起来可能有些神秘，但是，一旦掌握了它的语法，你就可以只用几行代码去解决非常复杂的任务，这就是SED的魅力所在。

### SED的典型用途

SED的用途非常广泛，例如：

- 文本替换
- 选择性的输出文本文件
- 从文本文件的某处开始编辑
- 无交互式的对文本文件进行编辑等

## 工作流

在本章中，我们将会探索SED是如何工作的，要想成为一个SED专家，你需要知道它的内部实现。SED遵循简单的工作流：**读取**，**执行**和**显示**，下图描述了该工作流：

![](https://oayrssjpa.qnssl.com/2016-10-31-14614770425007.jpg)

- **读取**： SED从输入流（文件，管道或者标准输入）中读取一行并且存储到它叫做 模式空间（**pattern buffer**） 的内部缓冲区
- **执行**： 默认情况下，所有的SED命令都在**模式空间**中顺序的执行，除非指定了行的地址，否则SED命令将会在所有的行上依次执行
- **显示**： 发送修改后的内容到输出流。在发送数据之后，**模式空间**将会被清空。
- 在文件所有的内容都被处理完成之前，上述过程将会重复执行

### 需要注意的几点

- 模式空间 （**pattern buffer**） 是一块活跃的缓冲区，在sed编辑器执行命令时它会保存待检查的文本
- 默认情况下，所有的SED命令都是在模式空间中执行，因此输入文件并不会发生改变
- 还有另外一个缓冲区叫做 保持空间 （**hold buffer**），在处理模式空间中的某些行时，可以用保持空间来临时保存一些行。在每一个循环结束的时候，SED将会移除模式空间中的内容，但是该缓冲区中的内容在所有的循环过程中是持久存储的。SED命令无法直接在该缓冲区中执行，因此SED允许数据在 **保持空间** 和 **模式空间**之间切换
- 初始情况下，**保持空间** 和 **模式空间** 这两个缓冲区都是空的
- 如果没有提供输入文件的话，SED将会从标准输入接收请求
- 如果没有提供地址范围的话，默认情况下SED将会对所有的行进行操作

### 示例

让我们创建一个名为 **quote.txt** 的文本文件，文件内容为著名作家*Paulo Coelho*的一段名言
    
    $ vi quote.txt 
    There is only one thing that makes a dream impossible to achieve: the fear of failure. 
     - Paulo Coelho, The Alchemist

为了理解SED的工作流，我们首先使用SED显示出quote.txt文件的内容，该示例与`cat`命令类似

    $ sed '' quote.txt
    There is only one thing that makes a dream impossible to achieve: the fear of failure.
    - Paulo Coelho, The Alchemist

在上面的例子中，quote.txt是输入的文件名称，两个单引号是要执行的SED命令。

首先，SED将会读取quote.txt文件中的一行内容存储到它的模式空间中，然后会在该缓冲区中执行SED命令。在这里，没有提供SED命令，因此对该缓冲区没有要执行的操作，最后它会删除模式空间中的内容并且打印该内容到标准输出，很简单的过程，对吧?

在下面的例子中，SED会从标准输入流接受输入

    $ sed '' 

当上述命令被执行的时候，将会产生下列结果

    There is only one thing that makes a dream impossible to achieve: the fear of failure. 
    There is only one thing that makes a dream impossible to achieve: the fear of failure.

在这里，第一行内容是通过键盘输入的内容，第二行是SED输出的内容。

> 从SED会话中退出，使用组合键`ctrl-D (^D)`

## 基础语法

本章中将会介绍SED中的基本命令和它的命令行使用方法。SED可以用下列两种方式调用：

    sed [-n] [-e] 'command(s)' files 
    sed [-n] -f scriptfile files

第一种方式在命令行中使用单引号指定要执行的命令，第二种方式则指定了包含SED命令的脚本文件。当然，这两种方法也可以同时使用，SED提供了很多参数用于控制这种行为。

让我们看看如何指定多个SED命令。SED提供了`delete`命令用于删除某些行，这里让我们删除第一行，第二行和第五行：

首先，使用`cat`命令显示文件内容

    $ cat books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

现在，使用SED移除指定的行，为了删除三行，我们使用`-e`选项指定三个独立的命令

    $ sed -e '1d' -e '2d' -e '5d' books.txt
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    6) A Game of Thrones, George R. R. Martin, 864

我们还可以将多个SED命令写在一个文本文件中，然后将该文件作为SED命令的参数，SED可以对模式空间中的内容执行文件中的每一个命令，下面的例子描述了SED的第二种用法

首先，创建一个包含SED命令的文本文件，为了便于理解，我们使用与之前相同的SED命令

    $ echo -e "1d\n2d\n5d" > commands.txt 
    $ cat commands.txt
    1d 
    2d 
    5d 

接下来构造一个SED命令去执行该操作

    $ sed -f commands.txt books.txt
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    6) A Game of Thrones, George R. R. Martin, 864 

### 标准选项

SED支持下列标准选项：

- **-n** 默认情况下，模式空间中的内容在处理完成后将会打印到标准输出，该选项用于阻止该行为

        $ sed -n '' quote.txt 
	
- **-e** 指定要执行的命令，使用该参数，我们可以指定多个命令，让我们打印每一行两次：

        $ sed -e '' -e 'p' quote.txt
        There is only one thing that makes a dream impossible to achieve: the fear of failure.
        There is only one thing that makes a dream impossible to achieve: the fear of failure.
         - Paulo Coelho, The Alchemist
         - Paulo Coelho, The Alchemist

- **-f** 指定包含要执行的命令的脚本文件

        $ echo "p" > commands
        $
        $ sed -n -f commands quote.txt
        There is only one thing that makes a dream impossible to achieve: the fear of failure.
         - Paulo Coelho, The Alchemist

### GNU选项

这些选项是GNU规范定义的，可能对于某些版本的SED并不支持。

- **-n**， **--quiet**, **--slient**：与标准的-n选项相同
- **-e script**，**--expression=script**：与标准的-e选项相同
- **-f script-file**， **--file=script-file**：与标准的-f选项相同
- **--follow-symlinks**：如果提供该选项的话，在编辑的文件是符号链接时，SED将会跟随链接
- **-i[SUFFIX]**，**--in-place[=SUFFIX]**：该选项用于对当前文件进行编辑，如果提供了SUFFIX的话，将会备份原始文件，否则将会覆盖原始文件
- **-l N**， **--line-lenght=N**：该选项用于设置行的长度为N个字符
- **--posix**：该选项禁用所有的GNU扩展
- **-r**，**--regexp-extended**：该选项将启用扩展的正则表达式
- **-u**， **--unbuffered**：指定该选项的时候，SED将会从输入文件中加载最少的数据，并且更加频繁的刷出到输出缓冲区。在编辑`tail -f`命令的输出，你不希望等待输出的时候该选项是非常有用的。
- **-z**，**--null-data**：默认情况下，SED对每一行使用换行符分割，如果提供了该选项的话，它将使用NULL字符分割行


## 循环

与其它编程语言类似，SED提供了用于控制执行流的循环和分支语句。

SED中的循环有点类似于**goto**语句，SED可以根据标签（label）跳转到某一行继续执行，在SED中，我们可以定义如下的标签：

    :label 
    :start 
    :end 
    :up

在上面的示例中，我们创建了四个标签。

要跳转到指定的标签，使用 **b** 命令后面跟着标签名，如果忽略标签名的话，SED将会跳转到SED文件的结尾。

> **b**标签用于无条件的跳转到指定的label。

为了更好地理解SED中的循环和分支，让我们创建一个名为books2.txt的文本文件，其中包含一些图书的标题和作者信息，下面的示例中会合并图书的标题和作者，使用逗号分隔。之后搜索所有匹配“Paulo”的行，如果匹配的话就在这一行的开头添加`-`，否则跳转到`Print`标签，打印出该行内容。

    $ cat books2.txt
    A Storm of Swords
    George R. R. Martin
    The Two Towers
    J. R. R. Tolkien
    The Alchemist
    Paulo Coelho
    The Fellowship of the Ring
    J. R. R. Tolkien
    The Pilgrimage
    Paulo Coelho
    A Game of Thrones
    George R. R. Martin
    
    $ sed -n '
    h;n;H;x
    s/\n/, /
    /Paulo/!b Print
    s/^/- /
    :Print
    p' books2.txt
    A Storm of Swords , George R. R. Martin
    The Two Towers , J. R. R. Tolkien
    - The Alchemist , Paulo Coelho
    The Fellowship of the Ring , J. R. R. Tolkien
    - The Pilgrimage , Paulo Coelho
    A Game of Thrones , George R. R. Martin

乍看来上述的代码非常神秘，让我们逐步拆解一下

- 第一行是`h;n;H;x`这几个命令，记得上面我们提到的 **保持空间** 吗？第一个`h`是指将当前模式空间中的内容覆盖到 **保持空间**中，`n`用于提前读取下一行，并且覆盖当前模式空间中的这一行，`H`将当前模式空间中的内容追加到 **保持空间** 中，最后的`x`用于交换模式空间和**保持空间**中的内容。因此这里就是指每次读取两行放到模式空间中交给下面的命令进行处理
- 接下来是 **s/\n/, /** 用于将上面的两行内容中的换行符替换为逗号
- 第三个命令在不匹配的时候跳转到**Print**标签，否则继续执行第四个命令
- **:Print**仅仅是一个标签名，而`p`则是print命令

为了提高可读性，每一个命令都占了一行，当然，你也可以把所有命令放在一行

    $ sed -n 'h;n;H;x;s/\n/, /;/Paulo/!b Print; s/^/- /; :Print;p' books2.txt 

> 关于`h`，`H`，`x`命令参考官方手册 [sed, a stream editor](https://www.gnu.org/software/sed/manual/sed.html#index-Copy-hold-space-into-pattern-space-168) *3.6 Less Frequently-Used Commands*节
    
## 分支

使用 **t** 命令创建分支。只有当前置条件成功的时候，**t** 命令才会跳转到该标签。

> **t**命令只有在前一个替换（s）命令执行成功的时候才会执行。

让我们看一些前面章节中的例子，与之前不同的是，这次我们将打印四个连字符"-"，而之前是一个。

    $ sed -n '
    h;n;H;x
    s/\n/, /
    :Loop
    /Paulo/s/^/-/
    /----/!t Loop
    p' books2.txt
    A Storm of Swords , George R. R. Martin
    The Two Towers , J. R. R. Tolkien
    ----The Alchemist , Paulo Coelho
    The Fellowship of the Ring , J. R. R. Tolkien
    ----The Pilgrimage , Paulo Coelho
    A Game of Thrones , George R. R. Martin

在上面的例子中，前面两行与上一节中讲的作用一致，第三行定义了一个*Loop*标签，接下来匹配存在“Paulo”的行，如果存在则在最前面添加一个*-*，接下来是我们这里的重点：

`/----/!t Loop`这一行首先检查上面添加`-`之后是否满足四个`-`，如果不满足则跳转到Loop继续执行第三行，这样不停的追加`-`，最后如果改行满足前面有四个`-`才继续往下执行。 
 
 为了提高可读性，我们将每一个SED命令独立一行，我们也可以在同一行中使用：
 
     sed -n 'h;n;H;x; s/\n/, /; :Loop;/Paulo/s/^/-/; /----/!t Loop; p' books.txt 

## 模式空间和保持空间

### 模式空间

对任何文件的来说，最基本的操作就是输出它的内容，为了实现该目的，在SED中可以使用**print**命令打印出模式空间中的内容。

首先创建一个包含行号，书名，作者和页码数的文件，在本文中我们将会使用该文件，你也可以创建任何其它的文件，但是这里我们就创建一个包含以下内容的文件

    $ vi books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho,288 
    6) A Game of Thrones, George R. R. Martin, 864

执行`p`命令

    $ sed 'p' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 
    6) A Game of Thrones, George R. R. Martin, 864

你可能会疑惑，为什么每一行被显示了两次？

你还记得SED的工作流吗？默认情况下，SED将会输出模式空间中的内容，另外，我们的命令中包含了输出命令`p`，因此每一行被打印两次。但是不要担心，SED提供了**-n**参数用于禁止自动输出模式空间的每一行的行为

    $ sed -n 'p' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

### 行寻址

默认情况下，在SED中使用的命令会作用于文本数据的所有行。如果只想将命令作用于特定的行或者某些行，则需要使用 **行寻址** 功能。

在SED中包含两种形式的行寻址：

- 以数字形式表示的行区间
- 以文本模式来过滤行

两种形式都使用相同的语法格式

    [address]command

#### 数字方式的行寻址

在下面的示例中SED只会对第3行进行操作

    $ sed -n '3p' books.txt 
    3) The Alchemist, Paulo Coelho, 197 

当然，我们还可以让SED输出某些行。在SED中使用逗号**,**分隔输出行号的范围，例如下面的代码会输出出2-5行的内容

    $ sed -n '2,5 p' books.txt 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288

特殊字符 **$** 代表了文件的最后一行，输出文件的最后一行

    $ sed -n '$ p' books.txt 
    6) A Game of Thrones, George R. R. Martin, 864 

也可以使用 **$** 指定输出的地址范围，下列命令输出第三行到最后一行

    $ sed -n '3,$ p' books.txt
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    5) The Pilgrimage, Paulo Coelho,288
    6) A Game of Thrones, George R. R. Martin, 864

SED还提供了另外两种操作符用于指定地址范围，第一个是加号（**+**）操作符，它可以与逗号（**,**）操作符一起使用，例如 `M, +n` 将会打印出从第`M`行开始的下`n`行。下面的示例将会输出第二行开始的下面四行

    $ sed -n '2,+4 p' books.txt 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

我们还可以使用波浪线操作符（**~**）指定地址范围，它使用`M~N`的形式，它告诉SED应该处理`M`行开始的每`N`行。例如，`50~5`匹配行号50，55，60，65等，让我们只输出文件中的奇数行

    $ sed -n '1~2 p' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    3) The Alchemist, Paulo Coelho, 197 
    5) The Pilgrimage, Paulo Coelho, 288

下面的代码则是只输出文件中的偶数行

    $ sed -n '2~2 p' books.txt 
    2) The Two Towers, J. R. R. Tolkien, 352 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    6) A Game of Thrones, George R. R. Martin, 864 

> 注意，如果使用的是Mac系统自带的sed命令，可能不支持**~**和**+**操作符。可以使用`brew install gnu-sed --with-default-names`重新安装GNU-SED。

#### 使用文本模式过滤器

SED编辑器允许指定文本模式来过滤出命令要作用的行。格式如下：

    /pattern/command

必须用正斜线将要指定的pattern封起来。sed编辑器会将该命令作用到包含指定文本模式的行上。

下面的示例中，将会输出所有作者为Paulo Coelho的书籍。

    $ sed -n '/Paulo/ p' books.txt
    3) The Alchemist, Paulo Coelho, 197 
    5) The Pilgrimage, Paulo Coelho, 288

模式匹配也可以与数字形式的寻址同时使用，在下面的示例会从第一次匹配到`Alchemist`开始输出，直到第5行为止。

    $ sed -n '/Alchemist/, 5 p' books.txt
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288

使用逗号（**,**）操作符指定匹配多个匹配的模式。下列的示例将会输出Two和Pilgrimage之间的所有行

    $ sed -n '/Two/, /Pilgrimage/ p' books.txt 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288

在使用文本模式过滤器的时候，与数字方式的行寻址类似，可以使用加号操作符 **+**，它会输出从当前匹配位置开始的某几行，下面的示例会从第一次Two出现的位置开始输出接下来的4行

    $ sed -n '/Two/, +4 p' books.txt
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

### 保持空间

在处理模式空间中的某些行时，可以用保持空间来临时保存一些行。有5条命令可用来操作保持空间

| 命令 | 描述 |
| --- | --- |
| h | 将模式空间复制到保持空间 |
| H | 将模式空间附加到保持空间 |
| g | 将保持空间复制到模式空间 |
| G | 将保持空间附加到模式空间 |
| x | 交换模式空间和保持空间的内容 |

关于保持空间这里就不在举例了，前面再**循环**部分讲解下面这个命令的时候我们已经对它的使用做了说明。

     $ sed -n 'h;n;H;x;s/\n/, /;/Paulo/!b Print; s/^/- /; :Print;p' books2.txt 

## 基本命令

本章将会讲解一些常用的SED命令，主要包括`DELETE`，`WRITE`，`APPEND`，`CHANGE`，`INSERT`，`TRANSLATE`，`QUIT`，`READ`，`EXECUTE`等命令。

### 删除命令  **d**

删除命令格式如下

    [address1[,address2]]d 

`address1`和`address2`是开始和截止地址，它们可以是行号或者字符串匹配模式，这两种地址都是可选的。

由命令的名称可以知道，**delete** 命令是用来执行删除操作的，并且因为SED是基于行的编辑器，因此我们说该命令是用来删除行的。注意的是，该命令只会移除模式空间中的行，这样该行就不会被发送到输出流，但原始内容不会改变。

    $ sed 'd' books.txt 

为什么没有输出任何内容？默认情况下，SED将会对每一行执行删除操作，这就是该命令为什么没有在标准输出中输出任何内容的原因。

下列命令只移除第四行

    [jerry]$ sed '4d' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

SED也接受使用逗号(,)分隔的地址范围。我们可以构造地址范围去移除N1到N2行，例如，下列命令将删除2-4行

    $ sed '2, 4 d' books.txt     
    1) A Storm of Swords, George R. R. Martin, 1216 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

SED的地址范围并不仅仅限于数字，我们也可以指定模式匹配作为地址，下面的示例会移除所有作者为Paulo Coelho的书籍

    $ sed '/Paulo Coelho/d' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    6) A Game of Thrones, George R. R. Martin, 864 

我移除所有以`Storm`和`Fellowship`开头的行
    
    $ sed '/Storm/,/Fellowship/d' books.txt  
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

### 文件写入命令 **w**

SED提供了 **write** 命令用于将模式空间中的内容写入到文件，与 **delete** 命令类似，下面是 **write** 命令的语法

    [address1[,address2]]w file 

**w** 指定是写命令， **file** 指的是存储文件内容的文件名。使用 **file** 操作符的时候要小心，当提供了文件名但是文件不存在的时候它会自动创建，如果已经存在的话则会**覆盖**原文件的内容。

下面的SED命令会创建文件books.txt的副本，在 **w** 和 **file** 之间只能有一个空格

    $ sed -n 'w books.bak' books.txt 

上述命令创建了一个名为 **books.bak** 的文件，验证一下两个文件的内容是否相同

    $ diff books.txt books.bak  
    $ echo $?

一旦执行上述的代码，你将会得到下列输出

    0

聪明的你可能已经想到了，这不就是 **cp** 命令做的事情吗！确实如此，**cp** 命令也做了同一件事情，但是SED是一个成熟的工具，使用它你可以只复制文件中的某些行到新的文件中，如下代码会存储文件中的奇数行到另一个文件
    
    $ sed -n '2~2 w junk.txt' books.txt  
    $ cat junk.txt 
    2) The Two Towers, J. R. R. Tolkien, 352 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    6) A Game of Thrones, George R. R. Martin, 864 

假设你希望存储所有独立作者的书到单独的文件。如果人工去做的话，肯定是非常无聊而且没有技术含量的，但是使用SED，你就有了更加聪明的方法去实现

    $ sed -n -e '/Martin/ w Martin.txt' -e '/Paulo/ w Paulo.txt' -e '/Tolkien/ w Tolkien.txt' books.txt    
    
    $ cat Martin.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    6) A Game of Thrones, George R. R. Martin, 864

    $ cat Paulo.txt
    3) The Alchemist, Paulo Coelho, 197 
    5) The Pilgrimage, Paulo Coelho, 288

    $ cat Tolkien.txt
    2) The Two Towers, J. R. R. Tolkien, 352 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432


### 追加命令 **a**

文本追加命令语法：

    [address]a\ 
    Append text 

在第四行之后追加一本新书：

    $ sed '4 a 7) Adultry, Paulo Coelho, 234' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    7) Adultry, Paulo Coelho, 234 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

在命令部分，4指的是行号，`a` 是append命令，剩余部分为要追加的文本。

在文件的结尾插入一行文本，使用 **$** 作为地址

    $ sed '$ a 7) Adultry, Paulo Coelho, 234' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 
    7) Adultry, Paulo Coelho, 234 
    
除了行号，我们也可以使用文本模式指定地址，例如，在匹配 `The Alchemist` 的行之后追加文本

    $ sed '/The Alchemist/ a 7) Adultry, Paulo Coelho, 234' books.txt  
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    7) Adultry, Paulo Coelho, 234 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

### 行替换命令 **c**

SED通过 **c** 提供了 **change** 和 **replace** 命令，该命令帮助我们使用新文本替换已经存在的行，当提供行的地址范围时，所有的行都被作为一组被替换为单行文本，下面是该命令的语法

    [address1[,address2]]c\ 
    Replace text

比如，替换文本中的第三行为新的内容

    $ sed '3 c 3) Adultry, Paulo Coelho, 324' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) Adultry, Paulo Coelho, 324 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

SED也接受模式作为地址

    $ sed '/The Alchemist/ c 3) Adultry, Paulo Coelho, 324' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) Adultry, Paulo Coelho, 324 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

多行替换也是支持的，下面的命令实现了将第4-6行内容替换为单行

    $ sed '4, 6 c 4) Adultry, Paulo Coelho, 324' books.txt  
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) Adultry, Paulo Coelho, 324

### 插入命令 **i**

插入命令与追加命令类似，唯一的区别是插入命令是在匹配的位置前插入新的一行。

    [address]i\ 
    Insert text 

下面的命令会在第四行前插入新的一行

    $ sed '4 i 7) Adultry, Paulo Coelho, 324' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    7) Adultry, Paulo Coelho, 324 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

### 转换命令 **y**

转换（Translate）命令 **y** 是唯一可以处理单个字符的sed编辑器命令。转换命令格式 如下

    [address]y/inchars/outchars/

转换命令会对inchars和outchars值进行一对一的映射。inchars中的第一个字符会被转换为outchars中的第一个字符，第二个字符会被转换成outchars中的第二个字符。这个映射过程会一直持续到处理完指定字符。如果inchars和outchars的长度不同，则sed编辑器会产生一 条错误消息。

    $ echo "1 5 15 20" | sed 'y/151520/IVXVXX/'
    I V IV XX

### 输出隐藏字符命令 **l**

你能通过直接观察区分出单词是通过空格还是tab进行分隔的吗？显然是不能的，但是SED可以为你做到这点。使用`l`命令（英文字母L的小写）可以显示文本中的隐藏字符（例如`\t`或者`$`字符）。

    [address1[,address2]]l 
    [address1[,address2]]l [len] 

为了测试该命令，我们首先将books.txt中的空格替换为tab。

    $ sed 's/ /\t/g' books.txt > junk.txt 

接下来执行`l`命令

    $ sed -n 'l' junk.txt
    1)\tStorm\tof\tSwords,\tGeorge\tR.\tR.\tMartin,\t1216\t$
    2)\tThe\tTwo\tTowers,\tJ.\tR.\tR.\tTolkien,\t352\t$
    3)\tThe\tAlchemist,\tPaulo\tCoelho,\t197\t$
    4)\tThe\tFellowship\tof\tthe\tRing,\tJ.\tR.\tR.\tTolkien,\t432\t$
    5)\tThe\tPilgrimage,\tPaulo\tCoelho,\t288\t$
    6)\tA\tGame\tof\tThrones,\tGeorge\tR.\tR.\tMartin,\t864$
    
使用`l`命令的时候，一个很有趣的特性是我们可以使用它来实现文本按照指定的宽度换行。

    $ sed -n 'l 25' books.txt
    1) Storm of Swords, Geor\
    ge R. R. Martin, 1216 $
    2) The Two Towers, J. R.\
     R. Tolkien, 352 $
    3) The Alchemist, Paulo \
    Coelho, 197 $
    4) The Fellowship of the\
     Ring, J. R. R. Tolkien,\
     432 $
    5) The Pilgrimage, Paulo\
     Coelho, 288 $
    6) A Game of Thrones, Ge\
    orge R. R. Martin, 864$

上面的示例中在`l`命令后跟了一个数字25，它告诉SED按照每行25个字符进行换行，如果指定这个数字为0的话，则只有在存在换行符的情况下才进行换行。

> `l`命令是GNU-SED的一部分，其它的一些变体中可能无法使用该命令。

### 退出命令 **q**

在SED中，可以使用`Quit`命令退出当前的执行流

    [address]q 
    [address]q [value]

需要注意的是，`q`命令不支持地址范围，只支持单个地址匹配。默认情况下SED会按照读取、执行、重复的工作流执行，但当它遇到`q`命令的时候，它会退出当前的执行流。

    $ sed '3 q' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197

    $ sed '/The Alchemist/ q' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197

`q`命令也支持提供一个value，这个value将作为程序的返回代码返回

    $ sed '/The Alchemist/ q 100' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197
    
    $ echo $? 
    100

### 文件读取命令 **r**

在SED中，我们可以让SED使用Read命令从外部文件中读取内容并且在满足条件的时候显示出来。

    [address]r file

需要注意的是，`r`命令和文件名之间必须只有一个空格。

下面的示例会打开*junk.txt*文件，将其内容插入到*books.txt*文件的第三行之后

    $ echo "This is junk text." > junk.txt 
    $ sed '3 r junk.txt' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    This is junk text. 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

> `r`命令也支持地址范围，例如*3, 5 r junk.txt*会在第三行，第四行，第五行后面分别插入*junk.txt*的内容

### 执行外部命令 **e**

如果你看过[三十分钟学会AWK][]一文，你可能已经知道了在AWK中可以执行外部的命令，那么在SED中我们是否也可以这样做？

答案是肯定的，在SED中，我们可以使用`e`命令执行外部命令

    [address1[,address2]]e [command]

下面的命令会在第三行之前执行*date*命令

    $ sed '3 e date' books.txt
    1) Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352
    2016年11月29日 星期二 22时46分14秒 CST
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    5) The Pilgrimage, Paulo Coelho, 288
    6) A Game of Thrones, George R. R. Martin, 864

另一个示例

    $ sed '3,5 e who' books.txt
    1) Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352
    mylxsw   console  Nov 29 19:30
    mylxsw   ttys000  Nov 29 22:45
    3) The Alchemist, Paulo Coelho, 197
    mylxsw   console  Nov 29 19:30
    mylxsw   ttys000  Nov 29 22:45
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    mylxsw   console  Nov 29 19:30
    mylxsw   ttys000  Nov 29 22:45
    5) The Pilgrimage, Paulo Coelho, 288
    6) A Game of Thrones, George R. R. Martin, 864

如果你仔细观察`e`命令的语法，你会发现其实它的*command*参数是可选的。在没有提供外部命令的时候，SED会将模式空间中的内容作为要执行的命令。

    $ echo -e "date\ncal\nuname" > commands.txt
    $ cat commands.txt
    date
    cal
    uname
    $ sed 'e' commands.txt
    2016年11月29日 星期二 22时50分30秒 CST
        十一月 2016
    日 一 二 三 四 五 六
           1  2  3  4  5
     6  7  8  9 10 11 12
    13 14 15 16 17 18 19
    20 21 22 23 24 25 26
    27 28 29 30
    
    Darwin
    
### 排除命令 **!**

感叹号命令（**!**）用来排除命令，也就是让原本会起作用的命令不起作用。

    $ sed -n '/Paulo/p' books.txt
    3) The Alchemist, Paulo Coelho, 197
    5) The Pilgrimage, Paulo Coelho, 288
    $ sed -n '/Paulo/!p' books.txt
    1) Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    6) A Game of Thrones, George R. R. Martin, 864

如上例所示，`p`命令原先是只输出匹配*Paulo*的行，添加`!`之后，变成了只输出不匹配*Paulo*的行。

    $ sed -n '1!G; h; $p' books.txt
    6) A Game of Thrones, George R. R. Martin, 864
    5) The Pilgrimage, Paulo Coelho, 288
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    3) The Alchemist, Paulo Coelho, 197
    2) The Two Towers, J. R. R. Tolkien, 352
    1) Storm of Swords, George R. R. Martin, 1216

上面的命令实现了类似`tac`命令类似的输出，将文本内容倒序输出。看起来有些晦涩难懂，分解一下却十分简单：

1. *1!G* 这句的意思是出了第一行之外，处理每一行的时候都将保持空间中的内容追加到模式空间（正序->倒序）
2. *h* 将模式空间中的内容复制到保持空间以备下一行匹配的时候追加到下一行的后面
3. *$p* 如果匹配到最后一行的话则输出模式空间中的内容
4. 上述步骤不断重复直到文本结束刚好将文件内容翻转了一次

### 多行命令

在使用sed编辑器的基础命令时，你可能注意到了一个局限。所有的sed编辑器命令都是针对**单行**数据执行操作的。在sed编辑器读取数据流时，它会基于**换行符**的位置将数据分成行。sed编辑器根据定义好的脚本命令一次处理一行数据，然后移到下一行重复这个过程。

幸运的是，sed编辑器的设计人员已经考虑到了这种情况，并设计了对应的解决方案。sed编辑器包含了三个可用来处理多行文本的特殊命令。

- **N**：将数据流中的下一行加进来创建一个多行组来处理
- **D**：删除多行组中的一行
- **P**：打印多行组中的一行

#### N - 加载下一行

默认情况下，SED是基于单行进行操作的，有些情况下我们可能需要使用多行进行编辑，启用多行编辑使用`N`命令，与`n`不同的是，`N`并不会清除、输出模式空间的内容，而是采用了追加模式。

    [address1[,address2]]N

下面的示例将会把*books2.txt*中的标题和作者放到同一行展示，并且使用逗号进行分隔

    $ sed 'N; s/\n/,/g' books2.txt
    A Storm of Swords ,George R. R. Martin
    The Two Towers ,J. R. R. Tolkien
    The Alchemist ,Paulo Coelho
    The Fellowship of the Ring ,J. R. R. Tolkien
    The Pilgrimage ,Paulo Coelho
    A Game of Thrones ,George R. R. Martin

#### D - 删除多行中的一行

sed编辑器提供了多行删除命令**D**，它只删除模式空间中的第一行。该命令会删除到换行符（含 换行符）为止的所有字符。

    $ echo '\nThis is the header line.\nThis is a data line.\n\nThis is the last line.' | sed '/^$/{N; /header/D}'
    This is the header line.
    This is a data line.
    
    This is the last line.

#### P - 输出多行中的一行

`P`命令用于输出`N`命令创建的多行文本的模式空间中的第一行。

    [address1[,address2]]P 

例如下面的命令只输出了图书的标题

    $ sed -n 'N;P' books2.txt
    A Storm of Swords
    The Two Towers
    The Alchemist
    The Fellowship of the Ring
    The Pilgrimage
    A Game of Thrones

### 其它命令

#### n - 单行next

小写的n命令会告诉sed编辑器移动到数据流中的下一文本行，并且覆盖当前模式空间中的行。

    $ cat data1.txt 
    This is the header line.
    
    This is a data line.
    
    This is the last line.
    $ sed '/header/{n ; d}' data1.txt 
    This is the header line.
    This is a data line.
    
    This is the last line.

上面的命令中，首先会匹配包含*header*的行，之后将移动到数据流的下一行，这里是一个空行，然后执行`d`命令对改行进行删除，所有就看到了这样的结果：第一个空行被删除掉了。

#### v - SED版本检查

`v`命令用于检查SED的版本，如果版本大于参数中的版本则正常执行，否则失败

    [address1[,address2]]v [version]

例如

    $ sed --version
    sed (GNU sed) 4.2.2

    $ sed 'v 4.2.3' books.txt
    sed: -e expression #1, char 7: expected newer version of sed
    
    $ sed 'v 4.2.2' books.txt
    1) Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    5) The Pilgrimage, Paulo Coelho, 288
    6) A Game of Thrones, George R. R. Martin, 864

## 特殊字符

在SED中提供了两个可以用作命令的特殊字符：**=** 和 **&** 。

### `=`命令

`=`命令用于输出行号，语法格式为

    [/pattern/]= 
    [address1[,address2]]=

例如为每一行输出行号

    $ sed '=' books2.txt
    1
    A Storm of Swords
    2
    George R. R. Martin
    ...
    
只为1-4行输出行号
    
    $ sed '1, 4=' books2.txt
    1
    A Storm of Swords
    2
    George R. R. Martin
    3
    The Two Towers
    4
    J. R. R. Tolkien
    The Alchemist
    Paulo Coelho
    The Fellowship of the Ring
    J. R. R. Tolkien
    The Pilgrimage
    Paulo Coelho
    A Game of Thrones
    George R. R. Martin

匹配Paulo的行输出行号

    $ sed '/Paulo/ =' books2.txt
    A Storm of Swords
    George R. R. Martin
    The Two Towers
    J. R. R. Tolkien
    The Alchemist
    6
    Paulo Coelho
    The Fellowship of the Ring
    J. R. R. Tolkien
    The Pilgrimage
    10
    Paulo Coelho
    A Game of Thrones
    George R. R. Martin

最后一行输出行号，这个命令比较有意思了，可以用于输出文件总共有多少行
    
    $ sed -n '$ =' books2.txt
    12

### `&`命令

特殊字符`&`用于存储匹配模式的内容，通常与替换命令`s`一起使用。

    $ sed 's/[[:digit:]]/Book number &/' books.txt
    Book number 1) Storm of Swords, George R. R. Martin, 1216
    Book number 2) The Two Towers, J. R. R. Tolkien, 352
    Book number 3) The Alchemist, Paulo Coelho, 197
    Book number 4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    Book number 5) The Pilgrimage, Paulo Coelho, 288
    Book number 6) A Game of Thrones, George R. R. Martin, 864

上述命令用于匹配每一行第一个数字，在其前面添加 *Book number* 。而下面这个命令则匹配最后一个数字，并修改为`Pages =`。其中`[[:digit:]]* *$`可能比较费解，这一部分其实是：*匹配0个或多个数字+0个或多个空格+行尾*。

    sed 's/[[:digit:]]* *$/Pages = &/' books.txt
    1) Storm of Swords, George R. R. Martin, Pages = 1216
    2) The Two Towers, J. R. R. Tolkien, Pages = 352
    3) The Alchemist, Paulo Coelho, Pages = 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, Pages = 432
    5) The Pilgrimage, Paulo Coelho, Pages = 288
    6) A Game of Thrones, George R. R. Martin, Pages = 864

## 字符串

### 替换命令 **s**

文本替换命令非常常见，其格式如下

    [address1[,address2]]s/pattern/replacement/[flags]

在前面我们使用的*books.txt*文件中，我们使用逗号“*,*”分隔每一列，下面的示例中，我们会使用替换命令将其替换为管道符“*|*”：

    $ sed 's/,/ |/' books.txt
    1) Storm of Swords | George R. R. Martin, 1216
    2) The Two Towers | J. R. R. Tolkien, 352
    3) The Alchemist | Paulo Coelho, 197
    4) The Fellowship of the Ring | J. R. R. Tolkien, 432
    5) The Pilgrimage | Paulo Coelho, 288
    6) A Game of Thrones | George R. R. Martin, 864

是不是觉得哪里不对？相信你已经发现，每一行的第二个逗号都没有被替换，只有第一个被替换了，确实如此，在SED中，使用替换命令的时候默认只会对第一个匹配的位置进行替换。使用`g`选项告诉SED对所有内容进行替换：

    $ sed 's/,/ | /g' books.txt
    1) Storm of Swords |  George R. R. Martin |  1216
    2) The Two Towers |  J. R. R. Tolkien |  352
    3) The Alchemist |  Paulo Coelho |  197
    4) The Fellowship of the Ring |  J. R. R. Tolkien |  432
    5) The Pilgrimage |  Paulo Coelho |  288
    6) A Game of Thrones |  George R. R. Martin |  864

> 如果对匹配模式（或地址范围）的行进行替换，则只需要在`s`命令前添加地址即可。比如只替换匹配*The Pilgrimage*的行：` sed '/The Pilgrimage/ s/,/ | /g' books.txt`

还有一些其它的选项，这里就简单的描述一下，不在展开讲解

- **数字n**: 只替换第n次匹配，比如`sed 's/,/ | /2' books.txt`，只替换每行中第二个逗号
- **p**：只输出改变的行，比如`sed -n 's/Paulo Coelho/PAULO COELHO/p' books.txt`
- **w**：存储改变的行到文件，比如`sed -n 's/Paulo Coelho/PAULO COELHO/w junk.txt' books.txt`
- **i**：匹配时忽略大小写，比如`sed  -n 's/pAuLo CoElHo/PAULO COELHO/pi' books.txt`

在执行替换操作的时候，如果要替换的内容中包含`/`，这个时候怎么办？很简单，添加转义操作符。

    $ echo "/bin/sed" | sed 's/\/bin\/sed/\/home\/mylxsw\/src\/sed\/sed-4.2.2\/sed/'
    /home/mylxsw/src/sed/sed-4.2.2/sed

上面的命令中，我们使用`\`对`/`进行了转义，不过表达式已经看起来非常难看了，在SED中还可以使用`|`，`@`，`^`，`!`作为命令的分隔符，所以，下面的几个命令和上面的是等价的

    echo "/bin/sed" | sed 's|/bin/sed|/mylxsw/mylxsw/src/sed/sed-4.2.2/sed|'
    echo "/bin/sed" | sed 's@/bin/sed@/home/mylxsw/src/sed/sed-4.2.2/sed@'
    echo "/bin/sed" | sed 's^/bin/sed^/home/mylxsw/src/sed/sed-4.2.2/sed^'
    echo "/bin/sed" | sed 's!/bin/sed!/home/mylxsw/src/sed/sed-4.2.2/sed!'

### 匹配子字符串

前面我们学习了替换命令的用法，现在让我们看看如何获取匹配文本中的某个子串。

在SED中，使用`\(`和`\)`对匹配的内容进行分组，使用`\N`的方式进行引用。请看下面示例

    $ echo "Three One Two" | sed 's|\(\w\+\) \(\w\+\) \(\w\+\)|\2 \3 \1|'
    One Two Three

我们输出了*Three*，*One*，*Two*三个单词，在SED的替换规则中，使用空格分隔了三小段正则表达式`\(\w\+\)`来匹配每一个单词，后面使用`\1`，，`\2`，`\3`分别引用它们的值。

## 管理模式

前面已经讲解过模式空间和**保持空间**的用法，在本节中我们将会继续探索它们的用法。

> 本部分内容暂未更新，请关注[程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，我将最先在Github的这个仓库中更新最新内容。

## 正则表达式

这一部分就是标准正则表达式的一些特殊字符以元字符，比较熟悉的请略过。

### 标准正则表达式

#### **^** 

匹配行的开始。

    $ sed -n '/^The/ p' books2.txt
    The Two Towers, J. R. R. Tolkien 
    The Alchemist, Paulo Coelho 
    The Fellowship of the Ring, J. R. R. Tolkien 
    The Pilgrimage, Paulo Coelho

#### **$**

匹配行的结尾

    $ sed -n '/Coelho$/ p' books2.txt 
    The Alchemist, Paulo Coelho 
    The Pilgrimage, Paulo Coelho

#### **.**

匹配单个字符（除行尾）

    $ echo -e "cat\nbat\nrat\nmat\nbatting\nrats\nmats" | sed -n '/^..t$/p'
    cat
    bat
    rat
    mat

#### **[]**

匹配字符集

    $ echo -e "Call\nTall\nBall" | sed -n '/[CT]all/ p'
    Call
    Tall

#### **[\^]**

排除字符集

    $ echo -e "Call\nTall\nBall" | sed -n '/[^CT]all/ p'
    Ball
    
#### **[-]**

字符范围。

    $ echo -e "Call\nTall\nBall" | sed -n '/[C-Z]all/ p' 
    Call 
    Tall
    
#### **\?** ，**\\+** ，\*

分别对应0次到1次，一次到多次，0次到多次匹配。

#### **{n}** ，**{n,}** ，**{m, n}**

精确匹配N次，至少匹配N次，匹配M-N次

#### **|**

或操作。

    $ echo -e "str1\nstr2\nstr3\nstr4" | sed -n '/str\(1\|3\)/ p' 
    str1
    str3

### POSIX兼容的正则

主要包含`[:alnum:]`，`[:alpha:]`，`[:blank:]`，`[:digit:]`，`[:lower:]`，`[:upper:]`，`[:punct:]`，`[:space:]`，这些基本都见名之意，不在赘述。

### 元字符

#### **\s**

匹配单个空白内容

    $ echo -e "Line\t1\nLine2" | sed -n '/Line\s/ p'
    Line 1 

#### **\S**

匹配单个非空白内容。

#### **\w** ， **\W**

单个单词、非单词。

## 常用代码段

### Cat命令 

模拟`cat`命令比较简单，有下面两种方式

    $ sed '' books.txt
    1) Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    5) The Pilgrimage, Paulo Coelho, 288
    6) A Game of Thrones, George R. R. Martin, 864
    
    $ sed -n 'p' books.txt
    1) Storm of Swords, George R. R. Martin, 1216
    2) The Two Towers, J. R. R. Tolkien, 352
    3) The Alchemist, Paulo Coelho, 197
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432
    5) The Pilgrimage, Paulo Coelho, 288
    6) A Game of Thrones, George R. R. Martin, 864

### 移除空行

    $ echo -e "Line #1\n\n\nLine #2" | sed '/^$/d'
    Line #1
    Line #2
    
### 删除连续空行

    $ echo -e "Line #1\n\n\nLine #2" | sed '/./,/^$/!d'
    Line #1
    
    Line #2

### 删除开头的空行

    $ echo -e "\nLine #1\n\nLine #2" | sed '/./,$!d'
    Line #1
    
    Line #2
    
### 删除结尾的空行

    $ echo -e "\nLine #1\nLine #2\n\n" | sed ':start /^\n*$/{$d; N; b start }'
    
    Line #1
    
    Line #2

### 过滤所有的html标签

    $ cat html.txt
    <html>
    <head>
        <title>This is the page title</title>
    </head>
    <body>
        <p> This is the <b>first</b> line in the Web page.
        This should provide some <i>useful</i> information to use in our sed script.
    </body>
    </html>                                                                                  
    $ sed 's/<[^>]*>//g ; /^$/d' html.txt
        This is the page title
         This is the first line in the Web page.
        This should provide some useful information to use in our sed script.

### 从C++程序中移除注释

有下面这样一个cpp文件

    $ cat hello.cpp
    #include <iostream> 
    using namespace std; 
    int main(void) 
    { 
       // Displays message on stdout. 
       cout >> "Hello, World !!!" >> endl;  
       return 0; // Return success. 
    }

执行下面的命令可以移除注释

    $ sed 's|//.*||g' hello.cpp
    #include <iostream>
    using namespace std;
    int main(void)
    {
    
       cout >> "Hello, World !!!" >> endl;
       return 0;
     }
     
### 为某些行添加注释

    $ sed '3,5 s/^/#/' hello.sh 
    #!/bin/bash 
    #pwd 
    #hostname 
    #uname -a 
    who 
    who -r 
    lsb_release -a

### 实现**Wc -l**命令

`wc -l`命令用于统计文件中的行数，使用SED也可以模拟该命令

    $ wc -l hello.cpp
           9 hello.cpp
    $ sed -n '$ =' hello.cpp
    9

### 模拟实现`head`命令

`head`命令用于输出文件中的前10行内容。

    $ head books2.txt
    A Storm of Swords
    George R. R. Martin
    The Two Towers
    J. R. R. Tolkien
    The Alchemist
    Paulo Coelho
    The Fellowship of the Ring
    J. R. R. Tolkien
    The Pilgrimage
    Paulo Coelho
    
使用SED中的`sed '10 q'`可以模拟它的实现

    $ sed '10 q' books.txt 
    A Storm of Swords 
    George R. R. Martin 
    The Two Towers 
    J. R. R. Tolkien 
    The Alchemist 
    Paulo Coelho 
    The Fellowship of the Ring 
    J. R. R. Tolkien 
    The Pilgrimage
    Paulo Coelho

### 模拟`tail -1`命令

`tail -1`输出文件的最后一行。

    $ cat test.txt
    Line #1 
    Line #2 
    
    $ tail -1 test.txt
    Line #2
    $ sed $ sed -n '$p' test.txt
    Line #2

### 模拟`Dos2unix`命令

在DOS环境中，换行符是使用**CR/LF**两个字符一起表示的，下面命令模拟了`dos2unix`命令转换这些换行符为UNIX换行符。

> 在GNU/Linux环境中，**CR/LF**通常使用"\^M"（不是简单的两个符号组合，请使用快捷键Ctrl+v,Ctrl+m输入）进行表示。

    $ echo -e "Line #1\r\nLine #2\r" > test.txt
    $ file test.txt
    test.txt: ASCII text, with CRLF line terminators
    $ sed 's/^M$//' test.txt > new.txt
    $ file new.txt
    new.txt: ASCII text
    $ cat -vte new.txt
    Line #1$
    Line #2$

### 模拟`Unix2dos`命令

    $ file new.txt
    new.txt: ASCII text
    $ sed 's/$/\r/' new.txt > new2.txt
    $ file new2.txt
    new2.txt: ASCII text, with CRLF line terminators
    
    $ cat -vte new2.txt
    Line #1^M$
    Line #2^M$

### 模拟`cat -E`命令

`cat -E`命令会在每一行的行尾输出一个*$*符号。

    $ echo -e "Line #1\nLine #2" | cat -E
    Line #1$
    Line #2$
    $ echo -e "Line #1\nLine #2" | sed 's|$|&$|'
    Line #1$
    Line #2$

> 注意，在Mac下不支持`cat -E`，可以直接使用sed代替

### 模拟`cat -ET`命令

`cat -ET`命令不仅对每一行的行尾添加*$*，还会将每一行中的TAB显示为*^I*。

    $ echo -e "Line #1\tLine #2" | cat -ET
    Line #1^ILine #2$
    $ echo -e "Line #1\tLine #2" | sed -n 'l' | sed 'y/\\t/^I/'
    Line #1^ILine #2$

### 模拟`nl`命令

命令`nl`可以为输入内容的每一行添加行号，记得之前介绍的`=`操作符吧，在SED中我们可以用它来实现与`nl`命令类似的功能。

    $ echo -e "Line #1\nLine #2" |nl
         1	Line #1
         2	Line #2
    $ echo -e "Line #1\nLine #2" | sed = |  sed 'N;s/\n/\t/'
    1	Line #1
    2	Line #2

上面的SED命令使用了两次，第一次使用`=`操作符为每一行输出行号，注意这个行号是独占一行的，因此使用管道符连接了第二个SED命令，每次读取两行，将换行符替换为Tab，这样就模拟出了`nl`命令的效果。

### 模拟`cp`命令

    $ sed -n 'w dup.txt' data.txt
    $ diff data.txt dup.txt
    $ echo $?
    0

### 模拟`expand`命令

`expand`命令会转换输入中的TAB为空格，在SED中也可以模拟它

    $ echo -e "One\tTwo\tThree" > test.txt
    $ expand test.txt > expand.txt
    $ sed 's/\t/     /g' test.txt > new.txt
    $ diff new.txt expand.txt
    $ echo $?
    0

### 模拟`tee`命令

`tee`命令会将数据输出到标准输出的同时写入文件。

    $ echo -e "Line #1\nLine #2" | tee test.txt  
    Line #1 
    Line #2 

在SED中，实现该命令非常简单

    $ sed -n 'p; w new.txt' test.txt
    One Two Three

### 模拟`cat -s`命令

`cat -s`命令会将输入文件中的多行空格合并为一行。

    $ echo -e "Line #1\n\n\n\nLine #2\n\n\nLine #3" | cat -s
    Line #1
    
    Line #2
    
    Line #3

在SED中实现

    $ echo -e "Line #1\n\n\n\nLine #2\n\n\nLine #3" | sed '1s/^$//p;/./,/^$/!d'
    Line #1
    
    Line #2
    
    Line #3

这里需要注意的是`/./,/^$/!d`这个命令，它的意思是匹配区间`/./`到`/^$`，区间的开始会匹配至少包含一个字符的行，结束会匹配一个空行，在这个区间中的行不会被删除。

### 模拟`grep`命令

    $ echo -e "Line #1\nLine #2\nLine #3" | grep 'Line #1'
    Line #1
    $ echo -e "Line #1\nLine #2\nLine #3" | sed -n '/Line #1/p'
    Line #1

### 模拟`grep -v`命令

    $ echo -e "Line #1\nLine #2\nLine #3" | grep -v 'Line #1'
    Line #2
    Line #3
    $ echo -e "Line #1\nLine #2\nLine #3" | sed -n '/Line #1/!p'
    Line #2
    Line #3

### 模拟`tr`命令

`tr`命令用于字符转换

    $ echo "ABC" | tr "ABC" "abc"
    abc
    $ echo "ABC" | sed 'y/ABC/abc/'
    abc

## 写在最后

看到这里，你肯定要吐槽了，不是说了三十分钟学会吗？你确定你能三十分钟学会？上次的[三十分钟学会AWK][]说三十分钟学会不靠谱，这次又不靠谱了。不好意思，这里的三十分钟其实只是为了吸引你的注意而已，只有在你已经用过SED并对它的一些特性有所了解的情况下三十分钟看完才是有可能的，毕竟那么多特殊字符，那么多命令需要记住。不过话说回来，看完之后你有收获吗？有的话，那本文的目的就达到了，之后用到SED的时候再回来参考一下就可以了。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star。

## 参考

- [Sed Tutorial](http://www.tutorialspoint.com/sed/index.htm)
- [Linux命令行与shell脚本编程大全（第3版）](http://www.ituring.com.cn/book/1698)


[三十分钟学会AWK]:https://aicode.cc/san-shi-fen-zhong-xue-huiawk.html


