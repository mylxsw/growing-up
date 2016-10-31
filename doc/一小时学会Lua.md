# Lua脚本语言入门

本文还 **未完成**，敬请期待...


## 基础部分

学习一门语言，亘古不变的入门课程是如何输出一个“Hello，world”字符串，好吧，让我们开始旅程。

    print("Hello, World")

就是这么简单，保存上述代码到`hello.lua`文件，执行下面的命令就可以了

    $ lua hello.lua
    
在Lua中执行的每一段代码，无论是作为一个文件还是交互模式中的一行，都叫做 **chunk**。连续的语句块之间不需要分隔符，当然，这也不是强制的，也可以使用分号`;`作为分隔符。下面的代码块都是合法的：

    a = 1
    b = a * 2
    
    a = 1;
    b = a * 2;
    
    a = 1; b = a * 2
    a = 1  b = a * 2 -- 这也是合法的，但是不推荐，最好加一个分号，更加明显一些

Lua支持在命令行中以交互模式运行，执行`lua`命令，就可以进入到Lua的交互模式，从交互模式退出可以有两种方式：

* ctrl+D (UNIX) 或 ctrl+Z (Windows)
* 执行标准库命令 os.exit()

> 使用 `lua -i prog` 可以让lua在执行完prog语句之后直接进入交互模式。

在交互模式下，可以使用`dofile()`函数执行外部脚本。该函数在测试某一段代码的时候是非常有用的。

    > dofile("hello.lua")
    Hello world
    > os.exit()

Lua的标识符是**大小写敏感**的，在使用标识符的时候与其它编程语言基本一致，下面这些保留字，无法作为标识符使用：

* and 
* break
* do
* else
* elseif
* end
* false
* goto
* for
* function
* if
* in
* local
* nil
* not
* or
* repeat
* return
* then
* true
* until
* while

> Lua中的标识符不建议使用下划线`_`开头，它们在Lua中保留有特殊用途。

Lua中的注释使用`--`开头，直到行尾结束，也可以使用`--[[`开始一个块注释，块注释以`]]`结束。

    --[[
    print(10)
    --]]
    
    ---[[
    print(10)
    --]]

上例中用了一点小技巧，第一个语句因为添加了块注释，因此中间代码不会被执行，而第二个，因为注释开始是`--`，因此作为行注释，下面的`]]`也被注释了，这样可以快速去掉块注释。

在Lua中，全局变量在使用的时候不需要任何声明，你可以直接使用，访问一个未经初始化的变量是不会造成错误的，只会得到一个`nil`值。

    > print(b)
    nil
    > b = 10
    > print(b)
    10

Lua解释器在加载一个Lua文件的时候，如果第一行是以`#`开头，它会忽略改行，因此，你可以在lua文件的开头一行添加`#!/usr/local/bin/lua`或者`#!/usr/bin/env lua`让该文件在Unix系统中作为脚本文件。

### 类型和值

Lua是一门动态类型语言，语言中并没有类型的定义，它的每个值都表明了其类型。在Lua中有八种基本类型：

* nil
* boolean
* number
* string
* userdata
* function
* thread
* table

使用`type()`函数可以获取某个值的类型（字符串表示）:

    > print(type("Hello"))
    string
    > print(type(123))
    number
    > print(type(true))
    boolean
    > print(type(print))
    function
    > print(type(nil))
    nil

> 注意，在Lua中，函数作为第一类值，可以像其他值一样进行维护。

#### Nil

nil是一种只有一个值 **nil** 的类型，它代表了没有值。在Lua中，全局变量在没有赋值的时候，默认就是nil，所有也可以通过给一个全局变量赋值为nil来删除它。

#### Booleans

布尔值有两个：**true** 和 **false**。在Lua中，只有 **false** 和 **nil** 在条件判断语句中作为 **false**，而其它任何值（包含0和空字符串）均为 **true**。

#### Numbers

数值类型代表了一个实数（双精度浮点型）。在Lua中，没有整数（integer）类型。

#### Strings

在Lua中，字符串的值是不可改变的，无法改变一个字符串中的某个字符。这一点与很多语言是一样的。字符串使用单引号或者双引号，它俩在Lua中是一样的，唯一的区别是，使用其中的一种可以在字符串中直接包含另外一种而不需要转义。

    > a = "one string"
    > b = string.gsub(a, "one", "another") -- 改变字符串的部分，实际上是创建一个新的字符串
    > print(a)
    one string
    > print(b)
    another string

使用长度操作符`#`可以获取字符串的长度，`..`用于连接两个字符串：

    > a = "Hello world"
    > print(#a)
    11
    > print ("hello" .. " world")
    hello world

Lua中，支持使用`[[`和`]]`限定一个长字符串，这一点与注释的使用是类似的。

    > page = [[
    >> Hello, world
    >> What your name?
    >> Where are you come from?
    >> ]]
    > print(page)
    Hello, world
    What your name?
    Where are you come from?
    
    > print(#page)
    54

> 有时候，在长字符串内部可能会包含`[[`或者`]]`，可以在符号的中间添加任意个`=`来实现，比如`[===[`，`]===]`。

#### 强制类型转换

Lua提供了在numbers和strings类型之间的自动类型转换。任何数字与字符串的运算将会尝试转换字符串为数字：

    > print ("10" + 2)
    12.0
    > print ("10 + 1")
    10 + 1
    > print ("-5.3e-10" * "2")
    -1.06e-09
    > print ("hello" + 1)
    stdin:1: attempt to perform arithmetic on a string value
    stack traceback:
           	stdin:1: in main chunk
           	[C]: in ?

同样，当Lua发现需要将数字作为字符串的时候，它也会自动进行转换：

    > print (10 .. 20)
    1020

> 注意，在对字符串和数字做比较的相等比较的时候，它们是不同的。比如`10=="10"`是 **false**。

如果希望显式的转换字符串为数字，可以使用`tonumber`函数，在无法转换的情况下返回`nil`。

    line = io.read()
    n = tonumber(line)
    if n == nil then
        error(line .. " is not a valid number ")
    else
        print(n * 2)
    end

转换数字为字符串有两种方式：

* 使用`tostring`函数，比如 `tostring(10)`
* 数字连接空字符，比如`10 .. ""`

#### Tables

Table是Lua中主要的（唯一的）数据结构化机制，并且是最强大的一种类型。它实现了关联数组。你可以认为table是一种动态分配的对象，程序只维护了对它的引用（指针）。Lua永远不会隐式的拷贝或者创建新的table。

要创建一个table，使用`constructor`表达式 `{}`。

    > a = {}
    > print(type(a))
    table
    > a["hello"] = "world"
    > print(a)
    table: 0x7fc210500600
    > print(a["hello"])
    world
    > print(a["key"]) -- 与全局变量一样，没有初始化的key对应的值为nil
    nil
    > print(a.hello) -- 这种方式是数组形式的一个语法糖
    world

使用table作为array或者list的时候，只需要使用整数类型的key即可，不需要实现声明长度，直接初始化需要的元素即可

    -- read 10 lines, storing them in a table 
    a = {} 
    for i = 1, 10 do
        a[i] = io.read() 
    end

> 注意，在Lua中约定，数组的开始元素下标是从 **1** 开始的！

当将table作为list使用的时候，通常我们需要知道列表的长度，通常的做法是在list中维护一个非数字的key来存储其长度。注意，任何未经初始化的索引都会返回`nil`，因此，可以使用来判断list的结尾。当然，这种方法只有在list中没有空洞的时候才有效，我们称这种没有空洞的list叫做 **sequence**。

> 所谓的空洞就是一个连续的索引中间包含几个没有值的索引。比如，索引为1234678，这里索引5上就有一个空洞（hole)。

对于 **sequence** 来说，Lua提供了操作符 `#` 来获取它的长度，它会返回最后一个索引值。

#### Functions

在Lua中，函数也是第一类值，程序可以使用变量存储函数，将函数作为参数或者是返回值。

#### Userdata 和 Threads

userdata类型用于存储任意的C数据到Lua变量中，处理赋值和相等判断之外，在lua中没有提供其它预定义的操作。

### 表达式

表达式部分各种编程语言都有很多相似性，因此这里就只记录需要注意的一些内容。

与其它语言一样，在Lua中，操作符`==`用于相等性判断，而操作符`~=`则用于判断不相等。

> 注意，`nil`只与它本身相等，在比较table和userdata的时候，Lua通过他们的引用进行比较，因此只有两个值是同一个对象的时候它们才相等。

在Lua中，逻辑操作符不能使用`&&`，`||`，`!`，而要使用 **and**，**or**，**not**。注意，所有的逻辑操作符都认为`false`和`nil`为**false**，其它的所有值都为**true**。

如果第一个操作数是false，则**and**操作符将返回第一个操作数，反之则返回第二个。而**or**操作符则在第一个操作数为true的时候返回第一个，反之第二个。

    > print (5 and 6)
    6
    > print (true and 6)
    6
    > print(4 and 5)
    5
    > print(nil and 5)
    nil
    > print(false and 5)
    false
    > print(4 or 5)
    4
    > print(nil or 5)
    5

> 一个很有用的使用方法是`x = x or v`，等价于`if not x then x = v end`。

在Lua中，字符串连接使用的是两个点操作符`..`。由于字符串是不可变的，因此，该操作符总是返回一个新创建的字符串。

    > a = "121"
    > b = "344"
    > print(a .. b)
    121344

之前已经说过，Lua中可以使用`#`操作符获取table或者字符串的长度，这里需要注意的是，获取table的长度的时候要确保table是一个sequence（没有空洞），否则获取的结果是错误的。

> 对没有数字key的sequence来说，其长度为0。

下面这种情况一定要注意:

    > a = {4, 2, 4, nil, nil}
    > print (#a)
    3

因为`nil`值是未初始化的，这里实际上等价于只有三个元素。

另一个操作符是`{}`，前面我们看到过，它是table的构造器，用于初始化一个table。

    days = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}
    w = {x=0, y=0, label="console"} 
    x = {math.sin(0), math.sin(1), math.sin(2)} 
    w[1] = "another field" -- add key 1 to table 'w' 
    x.f = w -- add key "f" to table 'x' 
    print(w["x"]) --> 0
    print(w[1]) --> another field 
    print(x.f[1]) --> another field 
    w.x = nil -- remove field "x"


### 语句

在Lua中也支持一系列的声明语句，大部分与C和Pascal类似。但是也有一些特殊的语句，比如多重赋值，本地变量声明等。

#### 赋值

在Lua中支持多重赋值，可以一次性将多个值赋给多个变量。

    a, b = 10, 2 * x
    x, y = y, x -- 交换变量x和y的值

这里需要注意的是，如果值和变量的个数不相同，会根据情况忽略值或者将变量设为nil。

    > a, b, c = 1, 2, 3, 4
    > print(a, b, c)
    1      	2      	3
    > x, y, z = 1, 2
    > print(x, y, z)
    1      	2      	nil

#### 本地变量和块

除了全局变量之外，Lua也支持本地变量（局部变量），使用**local**关键字创建本地变量。

    j = 10 -- 全局变量
    local i = 1 -- 局部变量

本地变量的作用域被限制在它所在的语句块（blocks）中。定义一个独立的语句块可以使用`do ... end`语句。

    local a, b = 1, 10 
    if a < b then 
        print(a) --> 1 
        local a -- '= nil' is implicit 
        print(a) --> nil 
    end -- ends the block started at 'then' 
    print(a, b) --> 1 10

> 在编程过程中，最好是使用局部变量，这样可以减少出错的几率。

#### 控制结构

Lua提供了一些语句用于控制程序的逻辑，使用**if**控制条件的执行，**while**，**repeat**和**for**用于迭代。语句**while**，**for**，**if**都有一个**end**终止符，而**repeat**则是**until**终止符。

##### if then else

    if a < 0 then 
        a = 0 
    end
    
    if a < b then 
        return a 
    else 
        return b 
    end
    
    if line > MAXLINES then 
        showpage() 
        line = 0 
    end

    if op == "+" then
        r = a + b 
    elseif op == "-" then
        r = a - b 
    elseif op == "*" then
        r = a*b
    elseif op == "/" then
        r = a/b 
    else
        error("invalid operation") end

##### while

local i = 1 
while a[i] do 
    print(a[i]) 
    i = i + 1 
end

##### repeat

    -- print the first non-empty input line 
    repeat
        line = io.read() 
    until line ~= "" 
    print(line)

##### for

**for**语句有两种变体：

* 数值for
* 通用for

数值for用于从start_exp到dest_exp，以step_exp为步长循环执行语句块，其语法为：

    for var = start_exp, dest_exp, step_exp do 
        ...
    end

例如:

    for i = 1, f(x) do print(i) end
    for i = 10, 1, -1 do print(i) end

> 如果希望一直循环没有上限，dest_exp可以设为`math.huge`。

对于for循环的三个控制参数，需要注意下面三点：

1. for循环的三个条件表达式只会在循环开始之前执行一次
2. 控制变量是一个local的变量，它是由for语句自动声明的，只在for的内部使用
3. 在循环内部不要改变控制变量的值

通用的for用于遍历迭代器函数返回的值。比如函数`pairs`用于遍历一个table。

    -- print all values of table 't' 
    for k, v in pairs(t) 
    do 
        print(k, v) 
    end

常见的迭代器函数有

* **io.lines** 迭代文件中的每一行
* **pairs** 迭代一个table
* **ipairs** 迭代一个sequence
* **string.gmatch** 迭代字符串中的单词

##### break，return，goto

**return**语句在函数中提前退出的时候，需要注意

    function foo () 
        return --<< SYNTAX ERROR 
        -- 'return' is the last statement in the next block 
        do return end -- OK 
        <other statements> 
    end

**goto**语句的label使用**::name::**这种格式，但是，也需要注意下面几点

* label遵循可见性原则，因此不能跳转到一个语句块内
* 无法跳出当前函数
* 不能跳入本地变量的作用域

在Lua中，只有`break`语句，并没有提供`continue`语句的支持，因此，如果需要使用这个功能的话，需要使用`goto`语句配合label：

    while some_condition do 
        ::redo:: 
        if some_other_condition then 
            goto continue 
        else if yet_another_condition then 
            goto redo 
        end 
        <some code> 
        ::continue:: 
    end

> 使用`goto`配合`::continue::`放到循环底部实现，同样，`::redo::`放到页面顶端实现Redo功能

### 函数

在Lua中，函数是抽象语句和表达式的主要机制。

在调用函数的时候，如果函数没有参数，则需要添加`()`表明这是一个调用，如果函数只有一个参数，并且这个参数是个字符串字面值或者table的构造器`{}`的话，括号也是可以忽略的。

    print "Hello World"  --> print("Hello World") 
    dofile 'a.lua' --> dofile ('a.lua')
    print [[a multi-line message]] --> print([[a multi-line message]]) 
    f{x=10, y=20}  --> f({x=10, y=20})
    type{}  --> type({})

在Lua中可以直接调用Lua或者C写的函数，在调用的时候，两者没有什么区别。

    -- add the elements of sequence 'a' 
    function add (a) 
        local sum = 0 
        for i = 1, #a do
            sum = sum + a[i] 
        end
        return sum 
    end

关于函数参数，如果某个参数调用的时候不提供，则该参数的值为nil。
  
    function f (a, b) print(a, b) end
    
    f(3) --> 3    nil
    f(3, 4)  --> 3    4
    f(3, 4, 5)  --> 3    4 (5 会被忽略)
        
在函数中，使用`n = n or 默认值`的形式可以为函数参数实现默认值的功能

    function incCount (n) 
        n = n or 1 
        count = count + n 
    end

#### 多返回值

在Lua中，函数返回值提供了一个非常便利的特性，支持返回多个值。

    > s, e = string.find("Hello lua users", "lua")
    > print(s, e)
    7      	9        

自定义函数中要返回多个结果的话使用`return`语句后面跟着多个结果，使用`,`分隔。

    function foo () 
        return "a", "b" 
    end
    
    x, y = foo()

这里需要注意的是，`return`语句后面的返回值并不需要括号。

在Lua中，有一个特别的多返回值函数`table.unpack`，该函数用于将一个数组中的值以多返回值的方式返回。该函数在函数调用的时候是很有用的。

    print(table.unpack{10, 23, 44})  -- 10 23 44
    a, b = table.unpack{10, 20, 44}
    print(a, b) -- 10 20
    f(table.unpack(a)) -- 变量a中的所有元素将依次作为函数f的参数

#### Variadic(可变参数)函数

Lua中的函数是可以有可变数量的参数的，函数参数使用`...`的时候，表明该函数是可变参数的，函数中可以使用`...`访问可变数量的参数。

    function add (...) 
        local s = 0 
        for i, v in ipairs{...} do
            s = s + v 
        end 
        return s 
    end
    
    print(add(3, 4, 10, 25, 12)) --> 54

    function foo1 (...) 
        print("calling foo:", ...) 
        return foo(...) 
    end

> 与`table.unpack`函数相反，可以使用`table.pack`函数将多个值转换为一个table，比如`local args = table.pack(...)`。

#### 命名参数

Lua中并没有提供对参数的命名支持，所有的参数都是基于位置的，因此下面的代码是**错误**的

    -- 错误的代码
    rename(old="temp.lua", new="temp1.lua")

但是，我们可以通过打包函数参数到一个table中来实现这个功能

    function rename (arg)
        return os.rename(arg.old, arg.new) 
    end
    
    rename{old="temp.lua", new="temp1.lua'}

### 深入理解函数

在Lua中，函数作为第一类值，它与字符串、数字等是一样的，在Lua中函数是没有名字的，所谓的函数名实际上是讲函数赋予了一个变量。

    foo = function (x) return 2*x end

#### 闭包

当我们在一个函数内部写另一个函数的时候，内部这个函数会拥有对外部函数所有本地变量的访问权限，我们称这个特性为 **词法作用域(lexical scoping)**。

    function newCounter () 
        local i = 0 
        return function () 
            i = i + 1 
            return i 
        end 
    end
    
    c1 = newCounter() 
    print(c1()) --> 1 
    print(c1()) --> 2

使用闭包，可以实现类似于沙箱的功能，用于创建一个安全的环境，限制不信任代码的执行。

    do
        local oldOpen = io.open 
        local access_OK = function (filename, mode)
            <check access> 
        end 
        io.open = function (filename, mode) 
            if access_OK(filename, mode) then
                return oldOpen(filename, mode) 
            else
                return nil, "access denied" 
            end
        end 
    end

#### 非全局函数

在Lua中，因为函数是第一类值，因此，它也支持在本地作用域中定义

    local f = function (<params>)
        <body> 
    end
    
    local function f (<params>)
        <body> 
    end

在使用局部函数进行递归操作的时候，要注意函数内是无法使用原来的函数名做为递归函数名的，因为此时该变量还未创建，因此，可以先使用local 定义一个变量名，然后再给其赋值。

    local fact 
    fact = function (n) 
        if n == 0 then 
            return 1 
        else 
            return n*fact(n-1) 
        end 
    end

#### 尾调用消除

另一个有趣的特性是Lua提供了对尾调用消除的支持。

尾调用其实就是一个调用的外衣，当一个函数在它的最后调用另一个函数的时候，在调用完该函数之后就没有其它的动作了。下面这个调用就是一个尾调用

    function f (x) return g(x) end

在尾调用的情景下，当前函数f并不需要再维持当前上下文中的任何信息了，当它返回的时候，直接就返回到了最外层，特别是在递归的时候，如果调用函数总是维持着当前调用的上下文信息，会无谓的占用大量的资源，而**尾调用消除**则可以不再占用栈空间。

在Lua中，只有`return func(args)`形式的调用时尾调用，当然，函数名称和参数都可以是复杂的表达式，Lua在函数调用开始前就会对它们完成计算，因此下面这个调用时尾调用：

    return x[i].foo(x[j] + a*b, i + j)

### 编译、执行和错误

#### 编译

之前已经见过`dofile`函数，它会执行外部文件，而该函数实际上只是一个辅助函数，实际的编译操作都是由`loadfile`来做的，该函数会从文件中读取一段Lua代码，然后将其编译为函数，但是并不执行它。另外，与`dofile`不同的是，`loadfile`并不会产生错误输出，而是返回错误码，因此我们的程序可以处理它。

    function dofile (filename) 
        local f = assert(loadfile(filename)) 
        return f() 
    end

如果加载文件失败，这里的`assert`函数会产生一个错误。

`load`函数与`loadfile`类似，只不过它用于将字符串编译为函数。

    > f = load("5 + 6")
    > f
    nil
    > f, err = load("5 + 6")
    > f
    nil
    > err
    [string "5 + 6"]:1: unexpected symbol near '5'
    > f = load("return 5 + 6")
    > f
    function: 0x7f7f78c00910
    > f()
    11

> `load`命令在加载指令时，对变量的引用是全局的，而不是本地的，因为load总是在全局环境中编译chunks。

    i = 32
    local i = 0
    f = load ("i = i + 1; print(i)")
    g = function () i = i + 1; print(i) end
    f() -- 33
    g() -- 1
    
使用`loadfile`加载文件之后，并不能直接使用其中的函数，因为他只是编译，使用的话需要先定义

    f = loadfile("foo.lua")
    print(foo)  --> nil
    f()  -- defines 'foo'
    foo("ok")  --> ok

#### 预编译

Lua在执行代码的时候，会先对代码进行预编译，我们可以预先编译其源码，之后直接运行预编译后的代码。

产生一个预编译的文件（二级制chunks）最简单的方法是使用**luac**。

    $ luac -o hello.lc hello.lua

Lua解释器可以执行产生的`hello.lc`文件

    $ lua hello.lc

#### 错误处理

在Lua中，产生任何错误都会让当前的chunk结束并且返回到应用。一些无法预期的条件下可能会产生错误，同时我们可以使用`error`函数手动触发一个错误。

    error(message, [,level])

比如

    print "enter a number:" 
    n = io.read("*n") 
    if not n then 
        error("invalid input") 
    end

函数`assert`用于检测它的第一个参数是否是`false`，如果是，则产生一个错误，否则返回这个参数的值。

程序出错的时候到底是返回错误码还是直接产生一个错误呢，可以遵循以下原则：如果这个异常可以轻松的避免，则应该产生一个错误，否则应该返回一个错误码。

对于大部分应用程序来说，使用Lua作为脚本语言的时候，并不需要在Lua中处理任何错误，应用程序会去处理这些。如果需要在Lua中处理错误的话，**使用pcall函数+匿名函数参数实现异常捕获**。

    local ok, msg = pcall(function () 
        <some code> 
        if unexpected_condition then 
            error() 
        end 
        <some code> 
        print(a[i]) -- potential error: 'a' may not be a table 
        <some code> 
    end) 
    
    if ok then -- no errors while running protected code
        <regular code> 
    else -- protected code raised an error: take appropriate action
        <error-handling code> 
    end

    local status, err = pcall(function () error({code=121}) end) 
    print(err.code) --> 121


### 协程


## 表和对象

### metatable

### 环境

Lua中所有的全局变量都保存在一个标准的table中，叫做 **全局环境** （*global environment*）。Lua中也会在全局变量`_G`中存储环境本身（也就是说`_G._G`等于`_G`）。

    -- 打印所有的全局变量
    for n in pairs(_G) do print(n) end

#### 名称为变量的全局变量

通常，我们可以通过使用下面的语句获取一个变量指定的全局变量的值

    value = loadstring("return " .. varname)()

不过上面这个语句需要单独编译拼接后的命令，这样显然效率不会很高，我们可以使用下面的方法实现同样的效果

    value = _G[varname]

#### 全局变量声明




### 模块和包

### 面向对象编程


## 标准库



---

参考：

- Programming In Lua 3

