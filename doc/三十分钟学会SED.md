# 三十分钟学会SED

本文还 未完成，敬请期待...

![](https://oayrssjpa.qnssl.com/2016-11-27-14802608583950.jpg)

## 概述

SED的英文全称是 **Stream EDitor**，它是一个简单而强大的文本解析转换工具，在1973-1974年期间由贝尔实验室的*Lee E. McMahon*开发，今天，它已经运行在所有的主流操作系统上了。

*McMahon*创建了一个通用的行编辑器，最终变成为了SED。SED的很多语法和特性都借鉴了**ed**编辑器。设计之初，它就已经支持正则表达式，SED可以从文件中接受类似于管道的输入，也可以接受来自标准输入流的输入。

SED由自由软件基金组织（FSF）开发和维护并且随着GNU/Linux进行分发，因此，通常它也称作 **GNU SED**。对于新手来说，SED的语法看起来可能是非常神秘的，但是，一旦掌握了它的语法，你就可以只用几行代码去解决非常复杂的任务，这就是SED的魅力所在。

<!-- more -->

### SED的典型用途

SED的用途非常广泛，例如：

- 文本替换
- 选择性的输出文本文件
- 从文本文件的某处开始编辑
- 无交互式的对文本文件进行编辑等

## 工作流

在本章中，我们将会探索SED是如何工作的，要想成为一个SED专家，你需要知道它的内部实现。SED遵循简单的工作流：**读取**，**执行**和**显示**，下图描述了该工作流：

![](https://oayrssjpa.qnssl.com/2016-10-31-14614770425007.jpg)

- **读取**： SED从输入流（文件，管道或者标准输入）中读取一行并且存储到它叫做 **pattern buffer** 的内部缓冲区
- **执行**： 所有的SED命令都在**pattern buffer**中顺序的执行，默认情况下，除非指定了行的地址，否则SED命令将会在所有的行上依次执行
- **显示**： 发送修改后的内容到输出流。在发送数据之后，**pattern buffer**将会被清空。
- 在文件所有的内容都被处理完成之前，上述过程将会重复执行

### 需要注意的几点

- 模式缓冲区是SED内部使用的可变的内存存储区
- 默认情况下，所有的SED命令都是在模式缓冲区中执行，因此输入文件并不会发生改变。GNU SED提供了修改输入文件的方法，我们将会在后续章节中介绍
- 还有另外一个缓冲区叫做 **hold buffer**，它也是SED内部的可变内存存储区，数据可以存储在该缓冲区中以备以后提取。在每一个循环结束的时候，SED将会移除模式缓冲区中的内容，但是该缓冲区中的内容在所有的循环过程中是持久存储的。SED命令无法直接在该缓冲区中执行，因此SED允许数据在 **hold buffer** 和 **pattern buffer**之间切换
- 初始情况下，**hold buffer** 和 **pattern buffer** 这两个缓冲区都是空的
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

首先，SED将会读取quote.txt文件中的一行内容存储到它的模式缓冲区中，然后会在该缓冲区中执行SED命令。在这里，没有提供SED命令，因此对该缓冲区没有要执行的操作，最后它会删除模式缓冲区中的内容并且打印该内容到标准输出，很简单的过程，对吧?

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

我们还可以将多个SED命令写在一个文本文件中，然后将该文件作为SED命令的参数，SED可以对模式缓冲区中的内容执行文件中的每一个命令，下面的例子描述了SED的第二种用法

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

- **-n** 默认情况下，模式缓冲区中的内容在处理完成后将会打印到标准输出，该选项用于阻止该行为

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

- 第一行是`h;n;H;x`这几个命令，记得上面我们提到的 **hold buffer** 吗？第一个`h`是指将当前模式缓冲区中的内容覆盖到 **hold buffer**中，`n`用于提前读取下一行，并且覆盖当前模式缓冲区中的这一行，`H`将当前模式缓冲区中的内容追加到 **hold buffer** 中，最后的`x`用于交换模式缓冲区和**hold buffer**中的内容。因此这里就是指每次读取两行放到模式缓冲区中交给下面的命令进行处理
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

## 模式缓冲区

对任何文件的来说，最基本的操作就是输出它的内容，为了实现该目的，在SED中可以使用**print**命令打印出模式缓冲区中的内容。让我们学习一下模式缓冲区是什么吧！

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

你可能会疑惑，为什么每一行被显示了两次呢？

你还记得SED的工作流吗？默认情况下，SED将会输出模式缓冲区中的内容，另外，我们的命令中包含了输出命令`p`，因此每一行被打印两次。但是不要担心，SED提供了**-n**参数用于禁止默认的模式缓冲区自动输出每一行的行为

    $ sed -n 'p' books.txt 
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

祝贺你，我们已经获得了期望的输出了，默认情况下，SED会对所有行执行命令，不过我们也可以强制SED只操作指定的行。例如，在下面的示例中SED只会对第三行进行操作

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

SED还提供了另外两种操作符用于指定地址范围，第一个是加号（**+**）操作符，它可以与逗号（,）操作符一起使用，例如 `M, +n` 将会打印出从第`M`行开始的下`n`行。下面的示例将会输出第二行开始的下面四行

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

## 模式匹配

在前面的章节中，我们学习了SED如何处理地址范围。本章将会介绍SED如何处理模式范围，模式范围可以是简单的文本或者复杂的正则表达式。下面的示例中，将会输出所有作者为Paulo Coelho的书籍。

    $ sed -n '/Paulo/ p' books.txt
    3) The Alchemist, Paulo Coelho, 197 
    5) The Pilgrimage, Paulo Coelho, 288

在上面的示例中，SED输出了匹配到Paulo的所有行。

模式匹配也可以与地址范围同时使用，在下面的示例会从第一次匹配到`Alchemist`开始输出，直到第5行为止。

    $ sed -n '/Alchemist/, 5 p' books.txt
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288

使用逗号（**,**）操作符指定匹配多个模式。下列的示例将会输出Two和Pilgrimage之间的所有行

    $ sed -n '/Two/, /Pilgrimage/ p' books.txt 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288

在使用模式匹配的时候，与地址范围类似，可以使用加号操作符 **+**，它会输出从当前匹配位置开始的某几行，下面的示例会从第一次Two出现的位置开始输出接下来的4行

    $ sed -n '/Two/, +4 p' books.txt
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

## 基本命令

本章将会讲解一些常用的SED命令，主要包括`DELETE`，`WRITE`，`APPEND`，`CHANGE`，`INSERT`，`TRANSLATE`，`QUIT`，`READ`，`EXECUTE`等命令。

### 删除命令 `DELETE`

删除命令格式如下

    [address1[,address2]]d 

`address1`和`address2`是开始和截止地址，它们可以是行号或者字符串匹配模式，这两种地址都是可选的。

由命令的名称可以知道，**delete** 命令是用来执行删除操作的，并且因为SED是基于行的编辑器，因此我们说该命令是用来删除行的。注意的是，该命令只会移除模式缓冲区中的行，这样该行就不会被发送到输出流，但原始内容不会改变。

    $ sed 'd' books.txt 

为什么没有输出任何内容呢？默认情况下，SED将会对每一行执行删除操作，这就是该命令为什么没有在标准输出中输出任何内容的原因。

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

### 写命令 `Write`

SED提供了 **write** 命令用于将模式缓冲区中的内容写入到文件，与 **delete** 命令类似，下面是 **write** 命令的语法

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


### 追加命令 `append`

对于任何文本编辑器来说，追加内容是最常用的操作之一，SED使用append命令提供了对该操作的支持，下面是append操作的语法

    [address]a\ 
    Append text 

我们在第四行之后追加一本新书，下面的命令展示了如何操作

    [jerry]$ sed '4 a 7) Adultry, Paulo Coelho, 234' books.txt 

执行上述命令之后，将会得到下列输出

    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    7) Adultry, Paulo Coelho, 234 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

在命令部分，4指的是行号，`a` 是append命令，剩余部分为要追加的文本。

让我们在文件的结尾插入一行文本，使用 **$** 作为地址，下面的例子描述了该实现

    [jerry]$ sed '$ a 7) Adultry, Paulo Coelho, 234' books.txt

在执行上述命令之后，你将会得到下列输出

    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 
    7) Adultry, Paulo Coelho, 234 
    
除了行号，我们也可以使用文本模式指定地址，例如，下面的例子中在匹配 `The Alchemist` 的行之后追加文本

    [jerry]$ sed '/The Alchemist/ a 7) Adultry, Paulo Coelho, 234' books.txt  
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    7) Adultry, Paulo Coelho, 234 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

注意，如果有多个匹配的模式的话，文本将会按照匹配的顺序依次追加，下列的例子描述了这种场景

    [jerry]$ sed '/The/ a 7) Adultry, Paulo Coelho, 234' books.txt 

执行上述代码之后，将会得到下列输出

    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    7) Adultry, Paulo Coelho, 234 
    3) The Alchemist, Paulo Coelho, 197 
    7) Adultry, Paulo Coelho, 234 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    7) Adultry, Paulo Coelho, 234 
    5) The Pilgrimage, Paulo Coelho, 288 
    7) Adultry, Paulo Coelho, 234 
    6) A Game of Thrones, George R. R. Martin, 864 

### 修改命令 `Change`

SED通过 **c** 提供了 **change** 和 **replace** 命令，改命令帮助我们使用新文本替换已经存在的行，当提供行的范围时，所有的行都被作为一组被替换为单行文本，下面是改命令的语法

    [address1[,address2]]c\ 
    Replace text

让我们使用一些其他文本替换第三行

    [jerry]$ sed '3 c 3) Adultry, Paulo Coelho, 324' books.txt

在执行该命令之后，将会得到如下输出

    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) Adultry, Paulo Coelho, 324 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864

SED也接受模式作为地址

    [jerry]$ sed '/The Alchemist/ c 3) Adultry, Paulo Coelho, 324' books.txt
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) Adultry, Paulo Coelho, 324 
    4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
    5) The Pilgrimage, Paulo Coelho, 288 
    6) A Game of Thrones, George R. R. Martin, 864 

使用单行替换多行文本

    [jerry]$ sed '4, 6 c 4) Adultry, Paulo Coelho, 324' books.txt  
    1) A Storm of Swords, George R. R. Martin, 1216 
    2) The Two Towers, J. R. R. Tolkien, 352 
    3) The Alchemist, Paulo Coelho, 197 
    4) Adultry, Paulo Coelho, 324

### 插入命令 `insert`







---
原文： [Sed Tutorial](http://www.tutorialspoint.com/sed/index.htm)



