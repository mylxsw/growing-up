[TOC]

[AIdea](https://github.com/mylxsw/aidea) 项目的前端代码是一个标准的 Flutter APP 项目，按照 Flutter 官方文档安装好 Flutter 环境就可以直接进行开发、调试和编译安装包了。

官方环境安装网站

https://flutter.cn/docs/get-started/install/macos

选择对应系统的版本，这里以 macOS 系统为例，带大家一步一步搭建一个完整的 Flutter 开发环境。


![](https://ssl.aicode.cc/mweb/20231127/17010068430925.jpg)


> 注意：如果使用的是 Apple 芯片的 macOS 系统，在安装前需要先确保 Rosetta 转译环境可用，执行下面的命令
> ```bash
> sudo softwareupdate --install-rosetta --agree-to-license
> ```


## 安装 Flutter SDK 

在 [macOS install](https://docs.flutter.dev/get-started/install/macos) 页面下载 Flutter SDK

![](https://ssl.aicode.cc/mweb/20231127/17010068737388.jpg)

将下载后的压缩包放在电脑的合适位置，如 `~/Workspace/library` 目录，然后解压

```bash
# 将下载后的软件包移动到 ~/Workspace/library 目录
mv ~/Downloads/flutter_macos_arm64_3.16.0-stable.zip ~/Workspace/library/
# 解压压缩包
unzip flutter_macos_arm64_3.16.0-stable.zip
# 现在可以删除下载的安装包了（可选）
rm -fr flutter_macos_arm64_3.16.0-stable.zip
```

现在 Flutter SDK 已经安装在了 `~/Workspace/library/flutter` 目录，我们设置一下环境变量，让 `flutter` 的可执行文件可以被直接执行。在 `~/.profile` 文件（或者是 `.bash_profile`、`.zshrc` 文件）中，添加下面的内容

```bash
# ⚠️ 注意：这里需要修改 `/Users/mylxsw/` 为你自己的 HOME 目录名称
export PATH=$PATH:/Users/mylxsw/Workspace/library/flutter/bin
```

然后，在命令行中执行 `flutter doctor` 命令，查看当前环境是否还需要安装其它的依赖

```bash
flutter doctor -v
```

![](https://ssl.aicode.cc/mweb/20231127/17010075980000.jpg)

其中我们看到前面有红色的 ✗ 的为有问题的项，对与初次安装的人来说，可能这里会显示有很多红色的 ✗，不过不要恐慌，我们只需要解决其中部分就可以了，而且解决起来大部分都非常简单。

- ① 一般安装好 SDK 后，第一项就是默认没有问题的，这里不在赘述
- ②⑥ 这里是 Android 工具的配置有问题，这里主要用于编译 Android 版本打包和在 Android 模拟器、真机上进行调试，后面我们会展开说，如果你不想打包 Android 安装包，这一项可以忽略
- ③④ 这两项主要用于配置 Xcode 环境，用于编译 iOS 和 macOS 的安装包以及模拟器、真机的调试和运行，如果不想打包 iOS 或者 macOS 安装包，则这一项可忽略
- ⑤ Chrome 浏览器，这一项用于 Web 端的开发，不需要 Web 端可忽略
- ⑦⑧ 这两个是开发工具，也就是你是用的 IDE，只要安装其中一个就可以了，我们这里使用 VS Code 进行 Flutter 开发，因此只要关注 ⑧ VS Code 的状态就可以了
- ⑨ 这里是指的网络是否是 OK 的，因为 Flutter 是 Google 开发的，国内要访问的话需要确保能够访问到国际互联网才行
    > 如果你无法访问国际互联网，可以看这里 [在中国网络环境下使用 Flutter](https://flutter.cn/community/china?tab=macos)。

## 准备开发环境

### 设置开发工具 - 以 VS Code 为例

#### 安装 VS Code 

我们首先需要下载安装 VS Code，已安装的请忽略。

下载安装地址在这里：https://code.visualstudio.com

![](https://ssl.aicode.cc/mweb/20231127/17010096019795.jpg)

#### 安装 Flutter 扩展

在 VS Code 中，进入到扩展标签页，搜索 Flutter，安装 Flutter 扩展。

![](https://ssl.aicode.cc/mweb/20231127/17010096988102.jpg)

### Web 开发环境

Web 开发环境非常简单，只需要下载安装最新版本的 Chrome 浏览器就可以了。下载地址在这里

https://www.google.cn/intl/zh-CN/chrome/

> ⚠️ 注意：必须是 Google 官方原版的 Chrome 浏览器，不能使用微软的 Edge 浏览器代替。

![](https://ssl.aicode.cc/mweb/20231127/17010098555851.jpg)

安装之后，再次执行 `flutter doctor` 命令，可以看到 ⑤ Chrome 浏览器这一项已经 OK 了，这时候我们就可以用来开发和编译 Web 版本的 App 了。

![](https://ssl.aicode.cc/mweb/20231127/17010099665797.jpg)

### Android 开发环境

Android 开发环境主要包含以下两件事情需要做

1. 安装 Android Studio 
2. 安装 Android 工具链

#### 安装 Android Studio

安装过程比较简单，只要下载好了安装包执行安装就可以了，下载地址

https://developer.android.com/studio?hl=zh-cn

> 可能需要连接国际互联网。

![](https://ssl.aicode.cc/mweb/20231127/17010102375852.jpg)

打开 Android Studio，初次进入会进入到 “Android Studio Setup Wizard” 页面。

![](https://ssl.aicode.cc/mweb/20231127/17010103767031.jpg)

下一步，选择自定义。

![](https://ssl.aicode.cc/mweb/20231127/17010108875906.jpg)

然后下面这个页面，我们保持默认就可以了，所有的组件都是默认选择安装的。

![](https://ssl.aicode.cc/mweb/20231127/17010109071899.jpg)


在 **License Agreement** 页面，同意所有协议之后，点击 Finish。

![](https://ssl.aicode.cc/mweb/20231127/17010104944953.jpg)

然后就开始进入到了组件下载页面，这里很多资源都是从 Google 下载的，因此需要能够访问国际互联网才行。

> 这里可能会耗时比较长，因为要从 Google 下载大量的软件包。

![](https://ssl.aicode.cc/mweb/20231127/17010105574478.jpg)


#### 安装配置 Android 工具链

打开 Android Studio，然后进入 SDK Manager 配置页面。

![](https://ssl.aicode.cc/mweb/20231127/17010120259581.jpg)

切换到 SDK Tools 标签页下，安装 **Android SDK Command-line Tools**。

![](https://ssl.aicode.cc/mweb/20231127/17010119684787.jpg)

然后执行下面的命令同意 Android 协议

```bash
flutter doctor --android-licenses
```

所有提示输入的地方都输入字符 `y`，然后回车。

![](https://ssl.aicode.cc/mweb/20231127/17010123803272.jpg)

最后，我们再次执行 `flutter doctor` 命令，可以看到 Android 环境已经就绪了。

![](https://ssl.aicode.cc/mweb/20231127/17010124513043.jpg)

### iOS、macOS 环境配置

#### 安装 Xcode

首先需要安装 Xcode，这个直接从 App Store 安装就可以了，安装完成之后，需要执行以下命令

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

然后再次执行下面的命令，同意 Xcode 的协议

```bash
sudo xcodebuild -license
```

![](https://ssl.aicode.cc/mweb/20231127/17010112166473.jpg)

窗口中提示输入的时候，输入 `agree` 后回车就可以了。

#### 安装 CocoaPods

CocoaPods 是一个用于 Swift 和 Objective-C Cocoa 项目的依赖管理器。在命令行中执行下面的命令安装 CocoaPods

```bash
sudo gem install -V cocoapods
```

错误处理：
 
1. 安装时出现 `The last version of drb (>= 0) to support your Ruby & RubyGems was 2.0.5` 错误。

    ![](https://ssl.aicode.cc/mweb/20231127/17010134211314.jpg)
    
    命令行执行以下命令 
    
    ```bash
    sudo gem install drb -v 2.0.5
    ```
    
    ![](https://ssl.aicode.cc/mweb/20231127/17010135301999.jpg)


 2. 安装时出现 `The last version of activesupport (>= 5.0, < 8) to support your Ruby & RubyGems was 6.1.7.6` 错误。

    ![](https://ssl.aicode.cc/mweb/20231127/17010134867596.jpg)

    命令行执行以下命令 
    
    ```bash
    sudo gem install activesupport -v 6.1.7.6
    ```
    
    ![](https://ssl.aicode.cc/mweb/20231127/17010135364869.jpg)

最后执行 `flutter doctor` 检查下安装状态

![](https://ssl.aicode.cc/mweb/20231127/17010136096720.jpg)

## 编译和运行

### 开发调试

在 VS Code 中打开 AIdea 项目所在的目录，执行 `flutter pub get` 命令安装依赖的软件包。

![](https://ssl.aicode.cc/mweb/20231127/17010138134446.jpg)

安装完成后，会发现 `lib` 目录中所有代码都不会有红色的错误提示了。

在编辑器的右下角，选择要运行的环境

![](https://ssl.aicode.cc/mweb/20231127/17010139307376.jpg)

我们在 ② 中，选择哪个环境，在运行时就会使用对应的环境运行了。

![](https://ssl.aicode.cc/mweb/20231127/17010140116789.jpg)

编辑器窗口中，打开 `lib/main.dart` 文件，然后右上角会出现运行的图标，如下图所示，点击即可启动 Chrome 浏览器，运行 Web 端。

![](https://ssl.aicode.cc/mweb/20231127/17010141161092.jpg)

下图是运行效果

![](https://ssl.aicode.cc/mweb/20231127/17010143991937.jpg)

### 打包

应用打包命令参考 Makefile 文件，主要包含以下

Web 端

```bash
flutter build web --web-renderer canvaskit --release 
```

macOS 端

```bash
flutter build macos --no-tree-shake-icons --release
```

iOS 端

```bash
flutter build ipa --no-tree-shake-icons --release 
```

Android 端

```bash
flutter build apk --release --no-tree-shake-icons
```

Windows 端

```bash
# 编译
flutter build windows --release
# 拷贝 sqlite3.dll 文件到项目运行目录
copy .\windows\sqlite3.dll .\build\windows\runner\Release\
```
