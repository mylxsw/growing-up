

本文中，我们将会使用 **Inno Setup** 这个软件来为 Flutter 应用创建 Windows 安装包。

## 安装 Inno Setup

首先安装 [Inno Setup](https://jrsoftware.org) 这个软件，
在 [Inno Setup Downloads](https://jrsoftware.org/isdl.php) 下载安装 Inno Setup，下载地址在这里

    https://jrsoftware.org/isdl.php

![](https://ssl.aicode.cc/mweb/20240304/17095247759345.jpg-maxsize800)


## 创建 Windows 安装包

### 编译 Flutter APP

使用命令行编译 Flutter APP 的 Windows 版本

```bash
flutter build windows --release
```

![](https://ssl.aicode.cc/mweb/20240304/17095277731139.jpg)

这里输出的 `build/windows/runner/Release` 目录就是编译好的软件目录。


### 创建安装包脚本

打开 Inno Setup，选择 `Create a new script file using the Script Wizard`

![](https://ssl.aicode.cc/mweb/20240304/17095250787481.jpg-maxsize800)

然后点击 “下一步”，在下面这个页面，填写应用的基本信息

![](https://ssl.aicode.cc/mweb/20240304/17095277116200.jpg-maxsize800)

下一步，修改应用文件夹名称

![](https://ssl.aicode.cc/mweb/20240304/17095277393615.jpg-maxsize800)

然后就进入到了比较关键的页面了，下面的页面中，选择应用包含的文件 

![](https://ssl.aicode.cc/mweb/20240304/17095253810191.jpg-maxsize800)
注意上图中 ①②③ 的说明：

- ① 选择应用的可执行文件，在项目目录的 `build/windows/runner/Release/应用名称.exe`

    ![](https://ssl.aicode.cc/mweb/20240304/17095278307472.jpg-maxsize800)

- ② 添加应用包含的 dll 文件，这里选择的是 Release 目录下最外层的 dll 文件

    ![](https://ssl.aicode.cc/mweb/20240304/17095255539711.jpg-maxsize800)
- ③ 选择 Release 目录下的 `data` 目录

    ![](https://ssl.aicode.cc/mweb/20240304/17095256708204.jpg-maxsize800)
    
    ![](https://ssl.aicode.cc/mweb/20240304/17095256983767.jpg-maxsize800)

    ⚠️ 在添加完目录后，需要选中目录，点击 `Edit`，设置目标子文件夹为 `data`
    
    ![](https://ssl.aicode.cc/mweb/20240304/17095279454126.jpg-thumb)

然后点击下一步，不需要关联文件类型

![](https://ssl.aicode.cc/mweb/20240304/17095259005896.jpg-maxsize800)

下一步，允许用户创建桌面快捷方式

![](https://ssl.aicode.cc/mweb/20240304/17095259359890.jpg-maxsize800)

接下来选择应该应用的文档，没有的话可以直接跳过

![](https://ssl.aicode.cc/mweb/20240304/17095259641581.jpg-maxsize800)

接下来就是选择安装模式，默认 “使用个管理员安装模式”，安装后，系统中所有用户都可以使用 APP

![](https://ssl.aicode.cc/mweb/20240304/17095259827763.jpg-maxsize800)

再次点击下一步，选择语言，然后下一步，这里设置输出文件夹，文件名，应用 Logo 等信息

![](https://ssl.aicode.cc/mweb/20240304/17095281473764.jpg-maxsize800)

最后一路点击下一步，直到完成，保存脚本为 install.iss 文件。

![](https://ssl.aicode.cc/mweb/20240304/17095263922724.jpg)


### 打包

打开创建好的 `install.iss` 文件，在 Inno Setup Complier 中，点击“编译”按钮，就可以开始应用的打包了。

![](https://ssl.aicode.cc/mweb/20240304/17095263011717.jpg-maxsize800)

输出下面的信息，说明打包完成了，在输出目录中就可以看到打包好的应用安装包了。

![](https://ssl.aicode.cc/mweb/20240304/17095283342193.jpg-maxsize800)


如下图所示，双击安装包就可以愉快的安装了

![](https://ssl.aicode.cc/mweb/20240304/17095283956578.jpg-maxsize800)


## 常见问题

### 如何设置默认勾选 “创建桌面快捷方式”

在 `install.iss` 文件中，将下图中的 Flags 设置为 `checkablealone`

![](https://ssl.aicode.cc/mweb/20240304/17095319031802.jpg)

效果

![](https://ssl.aicode.cc/mweb/20240304/17095319189400.jpg-maxsize800)


### 启动应用后，报错缺少 msvcp140.dll、vcruntime140.dll、vcruntime140_1.dll 文件

解决该问题，首先需要在开发机上（编译所用的 Windows 电脑），从 `C:/Windows/System32` 目录下找到这个文件，拷贝到项目的 `windows` 目录中

![](https://ssl.aicode.cc/mweb/20240304/17095271560406.jpg-maxsize800)

然后在 `windows/CMakeLists.txt` 文件中添加以下内容

```
install(FILES "msvcp140.dll" DESTINATION "${INSTALL_BUNDLE_LIB_DIR}"
  CONFIGURATIONS Profile;Release
  COMPONENT Runtime)

install(FILES "vcruntime140.dll" DESTINATION "${INSTALL_BUNDLE_LIB_DIR}"
  CONFIGURATIONS Profile;Release
  COMPONENT Runtime)

install(FILES "vcruntime140_1.dll" DESTINATION "${INSTALL_BUNDLE_LIB_DIR}"
  CONFIGURATIONS Profile;Release
  COMPONENT Runtime)
```


![](https://ssl.aicode.cc/mweb/20240304/17095273131465.jpg-maxsize800)

然后重新编译应用即可，需要注意的是，不要忘记在 `Inno Setup` 脚本 `install.iss` 文件中将这三个文件加进去。

![](https://ssl.aicode.cc/mweb/20240304/17095282502735.jpg)
