# 自动打包 

![auto_build](/assets/packing/auto_build.jpg)

## 应用构建

### 构建过程
- 配置（编译器确定当前系统环境）
- 确定标准库和头文件的位置
- 确定依赖关系 
- 头文件预编译(precompilation) 
- 预处理(preprocessing) 
- 编译(compilation) 
- 连接(Linking) 
- 打包

### 包的组成
- 一个`Mach-O`格式的二进制可执行文件(签名的数据就在这个二进制文件中)
- 一些资源文件。

### Mach-O可执行文件

Mach是一种操作系统内核。在Mach上，一种可执行的文件格是就是`Mach-O（Mach Object file format）`。iOS是从OS X演变而来，所以同样支持Mach-O格式的可执行文件。

`ipa`包实际上就是一个`zip`压缩包，解压之后会有一个`Payload`文件夹，其中有个`XXX.app`文件，它里面除了有个各种资源、图片等，有个和包名相同的文件——就是`二进制可执行文件`。
`file`命令查看文件类型：
```bash
file iXiao
iXiao: Mach-O universal binary with 2 architectures: [arm_v7:Mach-O executable arm_v7] [arm64]
iXiao (for architecture armv7):	Mach-O executable arm_v7
iXiao (for architecture arm64):	Mach-O 64-bit executable arm64
```
从上面看是支持arm7和arm64两种处理器架构的通用程序包，里面的格式是Mach-O。用Sublime打开部分如下：
```
cafe babe 0000 0002 0000 000c 0000 0009
0000 4000 0109 2df0 0000 000e 0100 000c
0000 0000 0109 8000 012a 4970 0000 000e
0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
``` 

开头的4个字节是`cafebabe`，这被称为“魔数”，反映文件的类型。
OS X上几个标识：

- `cafebabe`:就是跨处理器架构的通用格式
- `feedface`和`feedfacf`则分别是某一处理器架构下的Mach-O格式
- 脚本的就很常见了，比如`#!/bin/bash`开头的shell脚本。

### CodeResources文件

为了达到为所有文件设置签名，签名的过程中会在程序包中 `_CodeSignatue/CodeResources` 的`plist`文件中存储了被签名的程序包中所有文件的签名。

- 文件中不光包含了文件和它们的签名的列表、一系列规则，这些规则决定了哪些资源文件应当被设置签名。
- 10.9.5之后在 `CodeResources` 文件中会有4个不同区域，
  - `rules` 和 `files` 是老版本准备的，
  - `files2` 和 `rules2`是新版的代码签名准备的

  - 新版本中所有的代码文件和资源文件都必须设置签名。
  - 老版本程序包添加一个名为 `ResourceRules.plist` 的文件，在检查时被忽略。



## 打包

### 涉及工具
| 工具 |	作用 |
| --- | --- |
|PlistBuddy |	读写mobileprovision格式的文件，即可授权文件
|xcodebuild |	Xcode Project 构建
|security |	解码mobileprovision文件、获取可用签名列表
|codesign |	代码签名（此处值用来检查APP签名）


### xctool

1. 相比较xcodebuild输出的log杂乱，xctool更有结构
2. xctool有人性化的颜色输出
3. facebook声称xctool更快，据说能快2、3倍
4. 完全用Ojbective-C实现
xctool是可以使用[homebrew](http://brew.sh/)安装的，或者下[源码](https://github.com/facebook/xctool)然后运行 `xctool.sh`脚本，homebrew安装命令如下：

```
brew install xctool
```

### altool
- altool提交到App Store使用
- altool使用官方文档第38页
- altool 这个工具实际上是ApplicationLoader，打开Xcode-左上角Xcode-Open Developer Tool-Application Loader

altool的路径是：
```
/Applications/Xcode.app/Contents/Applications/Application\ Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Support/altool
```
使用时如报如下错误
```
altool[] *** Error: 
Exception while launching iTunesTransporter: Transporter not found at path: /usr/local/itms/bin/iTMSTransporter. 
You should reinstall the application.
```
建立软链
```
ln -s /Applications/Xcode.app/Contents/Applications/Application\ Loader.app/Contents/itms /usr/local/itms
```

### PlistBuddy
它是Mac下一个用来读写plist文件的工具，在/usr/libexec/下。
#### 使用
- PlistBuddy 使用冒号:来分割每个属性key的名字，例如下图假设需要获取name的值，那么冒号分割key的组成就是
命令：
```bash
/usr/libexec/PlistBuddy -c 'Print :Objects:0C14C6811E4964FA00F40247:List:2:name' $plistFile
```

## 编译
xcode的project的架构图
![](/assets/packing/auto_build2.png)

因为 PackageApplication 已经弃用，改用 xcodebuild -exportArchive ，所以 编译 也改用 archive，xcodebuild archive

打开终端，cd到你的工程位置，然后先试一下xcodebuild命令，
```
//xcrun
$ xcrun --version
xcrun version 35.

//xcodebuild
$ xcodebuild -version
Xcode 9.1
Build version 9B55

```
```
- xcodebuild -showsdks          -------   列出 Xcode 所有可用的 SDKs
- xcodebuild -list              -------   查看 project 中的 targets 和 configurations，或者 workspace 中 schemes
- xcodebuild -showBuildSettings -------   查看当前工程 build setting 的配置参数
```
