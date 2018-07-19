# 批量自动打包


## 项目需求
批量打上百个应用，项目的代码没有变动，应用有不同的icon，启动图，bundleID，第三方账号，和其他一些业务相关的差异。

### 思路
自动打包是基础，打包完之后可以根据母包重新签名生成相似的包，生成的包可以自动部署。只要攻克这三个点就能实现全自动化

1. 自动打包
2. 重新签名
      - 先自动打包生成一个母包，然后跑脚本对母包进行重新签名得到一个个子包。
      - bundleid替换
      - icon、启动图和第三方配置信息。

3. 分发部署


## 自动打包
![image](http://upload-images.jianshu.io/upload_images/1253942-f218f61e611122fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

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
- 脚本的就很常见了，比如#!/bin/bash开头的shell脚本。

### CodeResources文件

为了达到为所有文件设置签名，签名的过程中会在程序包中新建一个叫做 `_CodeSignatue/CodeResources` 的`plist`文件，文件中存储了被签名的程序包中所有文件的签名。

- 文件中不光包含了文件和它们的签名的列表、一系列规则，这些规则决定了哪些资源文件应当被设置签名。
- 10.9.5之后在 `CodeResources` 文件中会有4个不同区域，
  - `rules` 和 `files` 是老版本准备的，
  - `files2` 和 `rules2`是新版的代码签名准备的

  - 新版本中所有的代码文件和资源文件都必须设置签名。
  - 老版本程序包添加一个名为 `ResourceRules.plist` 的文件，在检查时被忽略。

更详细的介绍参见[《代码签名探析》](http://objccn.io/issue-17-2/)

## 自动打包

- 苹果自带的`xcodebuild`命令行工具
- [xctool](https://github.com/facebook/xctool)
  1. 相比较xcodebuild输出的log杂乱，xctool更有结构
  2. xctool有人性化的颜色输出
  3. facebook声称xctool更快，据说能快2、3倍
  4. 完全用Ojbective-C实现

### xctool

xctool是可以使用[homebrew](http://brew.sh/)安装的，或者下[源码](https://github.com/facebook/xctool)然后运行 `xctool.sh`脚本，homebrew安装命令如下：

```
brew install xctool
```
### PlistBuddy
脚本的开头有`PlistBuddy`命令，它是Mac下一个用来读写plist文件的工具，在/usr/libexec/下。


## 自动部署

通过自动部署，我们可以直接将包发到AppStore、fir\蒲公英这样的第三方平台、以及自己的服务器上。这里重点推荐**mattt**大神的——[**SHENZHEN**](https://github.com/nomad/shenzhen)。

### SHENZHEN的安装和使用

通过gem安装

```
$ gem install shenzhen
```

具体的用法可以参见[这里](https://github.com/nomad/shenzhen)，毕竟在中国，主要提下FIR和蒲公英已经上传AppStore的命令：

**FIR**

```
$ ipa distribute:fir -u USER_TOKEN -a APP_ID  
```

**蒲公英 (PGYER)**

```
$ ipa distribute:pgyer -u USER_KEY -a APP_KEY
```

USER信息到各自注册账号查找

**iTunes Connect Distribution**
```
$ ipa distribute:itunesconnect -a me@email.com -p myitunesconnectpassword -i appleid --upload
```

我们仍然是可以通过读取配置信息，来写个脚本跑部署，这部分就不再举例了，如果你不小心看到这个系列我觉得你应该会了，或者我们也可以相互商讨（毕竟我的blog人读的少 o(╯□╰)o）。实际上，我的同事已经实现了。

## 论持续化集成

虽然iOS好像能使用Jenkins进行持续化集成（好像我也用了一下Jenkins，貌似不是很好用，可能是我没有坚持用吧），但是通过这个序列的文章，其实我们自己就实现了一套持续化集成了。刚开始用蒲公英那会儿我把ipa包传给他们，他们就能放到平台给其他人测试，我觉得好神奇啊，后来想想，无非就是重新签名。持续化集成其实我的同事也实现了，我们用了一台服务器，定时的拉取代码，跑脚本，然后上传到测试服务器供人下载使用。只是公司内部推广不好，毕竟不是大厂也好像不是那么工程师文化，所以巴拉巴拉。


#### 参考：

1.  [SSL(https)中的对称加密和非对称加密](http://article.yeeyan.org/view/90729/174903)
2.  [RSA算法原理(一)](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
3.  [Bypassing OpenSSL Certificate Pinning in iOS Apps](http://chargen.matasano.com/chargen/2015/1/6/bypassing-openssl-certificate-pinning-in-ios-apps.html)
4.  [Understanding provisioning profiles and certificates](http://stackoverflow.com/questions/15238261/understanding-provisioning-profiles-and-certificates-in-ios)
5.  [Code Signing explained](http://www.pindh.com/2014/08/ios-development-code-signing-explained.html)
6.  [Mach-O可执行文件](http://blog.jobbole.com/51527/)
7.  [iOS开发中的各种证书](http://www.samirchen.com/ios-certificates/)
8.  [代码签名探析](http://objccn.io/issue-17-2/)
9.  [苹果证书和公钥私钥加密](http://blog.csdn.net/electronmc/article/details/45014591)
10.  [iOS8以后CodeSign失效问题](http://hennry.com/2015/03/fail-to-resign-ipa-since-ios8/)
11.  [iOS证书及ipa包重签名探究](http://www.olinone.com/?p=198)
12.  [ipa包部署网页安装](http://www.cocoachina.com/bbs/read.php?tid=94101)
13.  [iOS Code Signing: Under The Hood](http://www.raywenderlich.com/2915/ios-code-signing-under-the-hood)
14.  [How iOS developers use code signing to get their apps on iPhones](https://gigaom.com/2015/02/07/how-ios-developers-use-code-signing-to-get-their-apps-on-iphones/)
15.  [iOS Code Signing 学习笔记](http://foggry.com/blog/2014/10/16/ios-code-signing-xue-xi-bi-ji/)
16.  [苹果开发者账号那些事儿（二）](http://ryantang.me/blog/2013/09/03/apple-account-2/)
17.  [Inside Code Signing](https://www.objc.io/issues/17-security/inside-code-signing/)
18.  [公开密钥加密](https://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)
19.  [数字签名](https://zh.wikipedia.org/wiki/%E6%95%B8%E4%BD%8D%E7%B0%BD%E7%AB%A0)


