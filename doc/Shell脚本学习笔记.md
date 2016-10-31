# Shell脚本笔记

本文还 **未完成**，敬请期待...

## 基础知识备忘

### 执行算术运算

    val=`expr $a + $b`


### 运算符


| 符号 | 说明 | 示例 |
| --- | --- | --- |
| ! | 非运算 | [ ! false ] |
| -o  | 或运算 | [ $a -lt 20 -o $b -gt 20 ] |
| -a | 与运算 | [ $a -lt 20 -a $b -gt 20 ] |
| = | 相等检测 | [ $a = $b ] |
| != | 不相等检测 | [ $a != $b ] |
| -z | 字符串长度是否为0，为0则返回true | [ -z $a ] |
| -n | 字符串长度不为0， 不为0返回true | [ -n $a ] |
| str | 检测字符串是否为空，不为空返回true | [ $a ] |
| -b | 检测文件是否是块设备文件 | [ -b $file ] |
| -c | 检测文件是否是字符设备 | .. |
| -d | 检测文件是否为目录 | [ -d $file ] |
| -f | 检测文件是否为普通文件 | [ -f $file ] |
| -r | 检测文件是否可读 | .. |
| -w | 检测文件是否可写 | .. |
| -x | 检测文件是否可执行 | .. |
| -s | 检测文件是否为空 | .. |
| -e | 检测文件是否存在 | .. |

### 特殊变量


| 变量 | 含义 |
| --- | --- |
| $0 | 当前脚本的文件名 |
| $n | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数 |
| $# | 传递给脚本或函数的参数个数 |
| $* | 传递给脚本或函数的所有参数，所有参数被当做一个词，例如 "1 2 3" |
| $@ | 传递给脚本或函数的所有参数，每个参数当做一个词，用双引号包含，例如"1" "2" "3" |
| $? | 上个命令的退出状态，或函数的返回值 |
| $$ | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID |

### POSIX程序退出状态


| 状态码 | 含义 |
| --- | --- |
| 0 | 命令成功退出 |
| > 0 | 在重定向或者单词展开期间(~、变量、命令、算术展开以及单词切割)失败 |
| 1 - 125 | 命令不成功退出。特定的退出值的含义，有各个命令自行定义 |
| 126 | 命令找到了，但是文件无法执行 |
| 127 | 命令没有找到 |
| > 128 | 命令因收到信号而死亡 |

### 输入输出重定向

| 命令 | 说明 |
| ---- | ---- |
| command > file | 将输出重定向到 file。 |
| command > file | 将输出以追加的方式重定向到 file。 |
| n > file | 将文件描述符为 n 的文件重定向到 file。 |
| n >> file | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m | 将输出文件 m 和 n 合并。 |
| n <& m | 将输入文件 m 和 n 合并。 |
| << tag | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

### 文件包含

使用`.`或者`source`包含文件

    . filename
    source filename

> 被包含文件不需要有执行权限

### 使用函数

下面是函数基本结构
    
    # 直接使用函数名称即可，前面可选的添加function function_name
    function_name(){
    
        # 使用$#,$*,$@以及$0-$n获取函数名以及其它参数
        val="$1"
        
        # 带返回值，可选，返回值只能是整数（状态码）
        # 如果要返回结果的话，使用全局变量
        return $val
    }

函数调用
    
    # 调用函数直接使用函数名称即可，后面可以空格隔开使用多个参数
    function_name 1 2 3 4
    # 使用$?变量获取返回值
    ret=$?

## 常用代码片段

### 列出目录下的所有文件

下面的代码列出了downloads目录下所有的xlsx文件，这里要注意的是，如果要列出所有文件，使用`"$watch_dir"/*`这种形式。

    watch_dir="/Users/mylxsw/Downloads"
    # 避免目录下没有匹配文件时返回带有*的结果
    shopt -s nullglob
    for file in "$watch_dir"/*.xlsx
    do
        echo $file
    done
    
    for file in "$watch_dir"/*/    # 列出所有目录
    do 
        echo $file
    done

> 参考 [How to get the list of files in a directory in a shell script?](http://stackoverflow.com/questions/2437452/how-to-get-the-list-of-files-in-a-directory-in-a-shell-script) 和 [Bash Shell Loop Over Set of Files](http://www.cyberciti.biz/faq/bash-loop-over-file/)

### 从标准输入循环读取

    while read line
    do
        if [ $line != 'EOF' ]
        then
            echo $line
        fi
    done

### 日期时间处理

#### 获取当前日期

    # 输出: 20161023
    echo $(date +%Y%m%d)

#### 时间比较

时间比较可以先转换为UNIX时间戳，然后直接比较时间戳大小即可。

    time1=`date +%s`
    time2=`date -d '2016-10-25 17:20:13' +%s`
    
    if [ $time1 -gt $time2 ]; then
        echo "time1 > time2"
    else
        echo "time1 < time2"
    fi

