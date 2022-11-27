# Java 字符串 split 的一个反直觉陷阱

[TOC]

最近生产环境遇到一个奇怪的数组下标越界报错，如下图代码所示，我们可以肯定的是 `fieldName` 变量不为空（不是空字符串，也不是 `null`），但是代码执行到读取 `names[0]` 变量的时候，抛出了一个 **数组下标越界** （`java.lang.ArrayIndexOutOfBoundsException`） 的异常。

![](https://ssl.aicode.cc/mweb/2022-11-27-16695298707468.jpg?imageView2/2/w/800)


异常信息如下图所示

![](https://ssl.aicode.cc/mweb/2022-11-27-16695299980772.jpg)


问题很简单，我们对一个字符串执行 `split` 方法之后，以过往其它编程语言（Go、PHP、Javascript、Dart 等）的使用经验来看，即使字符串为空，即使没有匹配到分隔符，在返回值数组中也会包含一个当前字符串的值。但是这里却抛出了 `ArrayIndexOutOfBoundsException`，难道 `split` 方法的返回值可能为空数组？

最终经过排查发现，在上述代码段中，当 `fieldName` 的值为 `"~"` 的时候，我们访问 `names[0]` 就会抛出 `ArrayIndexOutOfBoundsException`，为什么会这样呢？


本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 问题

在 Java 中，如果执行下面这段代码，直觉上你认为会输出什么？

```java
String str = "~";
String []arr = str.split("~");

System.out.println(arr.length);
```

如果你有其他编程语言的经验，可能直觉上会觉得这里输出的应该是 **2**，但是遗憾的是，这里输出的是 **0**，变量 `arr` 是个空数组。

这里不禁怀疑自己之前的记忆是不是有偏差，于是我又使用其它语言来尝试复现这个问题。

## 不同语言中 split 的行为

我总结了一个表格，说明了不用语言不同的行为，这里对比的是执行 `split` 函数/方法后返回数组的长度：

| 语言\函数  | `"".split("")` | `"~".split("~")` | `"~~".split("~")` | `"".split("~")` | `"~123".split("~")` |
| ---------- | -------------- | ---------------- | :---------------- | :-------------- | :------------------ |
| Javascript | 0 | 2 | 3 | 1 | 2 |
| PHP        | 0 | 2 | 3 | 1 | 2 |
| Dart       | 0 | 2 | 3 | 1 | 2 |
| Golang     | 0 | 2 | 3 | 1 | 2 |
| Scala      | 1 | 0 | 0 | 1 | 2 |
| Java       | 1 | 0 | 0 | 1 | 2 |


### Javascript

首先是 Javascript，在浏览器的控制台上直接执行，得到了下面的结果

```javascript
"".split("")
"~".split("~")
"~~".split("~")
"".split("~")
"~123".split("~")
```

执行结果

![](https://ssl.aicode.cc/mweb/2022-11-27-16695300360838.jpg?imageView2/2/w/400)


跟我的直觉是一致的，同样的情况，这里返回的是 **2**。

### PHP

在 PHP 中，我使用了 `mb_split` 函数，该函数用于对多字节字符串进行分割

![](https://ssl.aicode.cc/mweb/2022-11-27-16695300562588.jpg)


执行结果如下

![](https://ssl.aicode.cc/mweb/2022-11-27-16695302942502.jpg?imageView2/2/w/600)

执行结果跟我的直觉也是一致的，同样的情况，这里返回的是 **2**。

### Dart

然后是 Google 的 Dart，这是一门主要用于使用 Flutter 来开发跨平台应用的编程语言，代码如下

```dart
void main() {
    print("".split('').length); // 0
    print("~".split('~').length); // 2
    print("~~".split('~').length); // 3
    print("".split('~').length); // 1
    print("~123".split('~').length); // 2
}
```

执行结果

![](https://ssl.aicode.cc/mweb/2022-11-27-16695303656914.jpg)


同样，`"~".split("~")` 也是返回了两个值。

### Golang

在 Golang 中，执行结果依旧是符合直觉的，返回的是 **2**。

```go
package main

import(
    "strings"
    "fmt"
)

func main() {
    printStrs(strings.Split("", "")) // 0 []
    printStrs(strings.Split("~", "~")) // 2 ["", "", ]
    printStrs(strings.Split("~~", "~")) // 3 ["", "", "", ]
    printStrs(strings.Split("", "~")) // 1 ["", ]
    printStrs(strings.Split("~123", "~")) // 2 ["", "123", ]
}

func printStrs(s []string) {
    fmt.Print(len(s), " [")
    for _, item := range s {
        fmt.Printf(`"%s", `, item)
    }

    fmt.Print("]\n")
}
```

执行结果

![](https://ssl.aicode.cc/mweb/2022-11-27-16695303487239.jpg)


### Scala

然后，我又尝试了 Scala，发现在 Scala 中， `split` 的行为有些不一样了。

```scala
"".split("").length
"~".split("~").length
"~~".split("~").length
"".split("~").length
"~123".split("~").length
```

![](https://ssl.aicode.cc/mweb/2022-11-27-16695304015889.jpg?imageView2/2/w/800)


代码 `"~".split("~")` 返回的是 **空数组**，与在 Java 中我们遇到的问题如出一辙。

### Java

最后，我又用 Java 执行了同样的代码

```java
package example;
import org.junit.Test;

public class ExampleTest {
  @Test
  public void testSplit() {
    printStrings("".split("")); // 1 ["", ]
    printStrings("~".split("~")); // 0 []
    printStrings("~~".split("~")); // 0 []
    printStrings("".split("~")); // 1 ["", ]
    printStrings("~123".split("~")); // 2 ["", "123", ]
  }
  
  private void printStrings(String[] strings) {
    System.out.print(strings.length + " [");
    for (String str : strings) {
      System.out.printf("\"%s\", ", str);
    }
    System.out.println("]");
  }
}
```


执行结果

![](https://ssl.aicode.cc/mweb/2022-11-27-16695304339575.jpg?imageView2/2/w/600)

结果与 Scala 是一致的，同时也解释了为什么我们会遇到 `ArrayIndexOutOfBoundsException` 的问题。

## 原因

翻阅了 Java 的 API 文档，发现原来 Java 中的 `split` 方法确实跟其它语言是不一样的，这一点我们特别容易忽略

![](https://ssl.aicode.cc/mweb/2022-11-27-16695305615890.jpg)


如果分隔符表达式与字符串不匹配，则返回原始字符串作为数组的唯一值，这也就解释了 

```java
"".split("") // 1 [""]
"".split("~") // 1 [""]
```

如果分隔符表单式与字符串的开始字符就已经匹配了，则返回值中第一个元素会被设置为 `""` 

```java
"~123".split("~") // 2 ["", "123"]
```

如果 `limit` 参数为 **0**，也就是 `split(String regex)` 方法，则匹配结果末尾的所有空字符串 `""` 都会被丢弃，也就解释了下面两段代码

```java
"~".split("~") // 0 []
"~~".split("~") // 0 []
```

![](https://ssl.aicode.cc/mweb/2022-11-27-16695306149805.jpg)



然后我又翻阅了 Scala 的官方文档，Scala 和 Java 的行为是一致的。

![](https://ssl.aicode.cc/mweb/2022-11-27-16695307067889.jpg)


## 总结

在 Java 中使用字符串的 `split` 方法，一般情况下的行为是和其他编程语言是一致的，但在一些边界条件下，也有一些不一致的地方，这一点是我们应该注意的，这也提醒了我们，不要想当然的认为不同语言，同名函数（方法）的功能是完全一致的，当我们遇到一些奇奇怪怪的问题时，多看官方文档才是硬道理。


本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。
