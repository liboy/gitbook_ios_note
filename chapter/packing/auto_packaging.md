# 批量自动打包


## 项目需求
批量打上百个应用，项目的代码没有变动，应用有不同的icon，启动图，bundleID，第三方账号，和其他一些业务相关的差异。

### 思路
1. 自动打包
2. 重新签名
      - 先自动打包生成一个母包，然后跑脚本对母包进行重新签名得到一个个子包。
      - bundleid替换
      - icon、启动图和第三方配置信息。

3. 分发部署


## 对称加密和非对称加密
![image](http://upload-images.jianshu.io/upload_images/1253942-5aa0f5f1990499b5.jpg)
*   对称加密（symmetric cryptography）：加密和解密用的是同一个秘钥
*   非对称加密（asymmetric cryptography）：加密和解密是不同的钥匙

客户端（Client）和服务器（Server）进行通讯，Client和Server约定好相同的一把秘钥，Client发送的明文通过这把秘钥进行加密，Server在收到这段加密后的密文后通过事先约定好的那边秘钥进行解密得到明文。理论上只要保证秘钥不被泄露就可以保证安全，但是实际上这种方式很不安全，如果秘钥被破解，又恰好服务器只用了这一个秘钥（这可能是最糟糕的情况），那么服务器和其他的客户端之间的通讯基本上就是完全暴露了。这个例子说的是对称加密。
![对称加密](http://upload-images.jianshu.io/upload_images/1253942-05697d31c8455025.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有一种情况，比上面的想法更加出色一点。每个端都生成一个“私钥-公钥”对，私钥是自己保管，公钥可以随便分享，用公钥可以解开私钥，用私钥可以解开公钥。例如服务端生成了一个“私钥-公钥”对，自己保留了一份私钥，把公钥给客户端，客户端对发送的消息通过公钥进行加密，服务端在收到这个公钥后用自己的私钥进行解密还原得到明文。

![非对称加密](http://upload-images.jianshu.io/upload_images/1253942-07b9e270cc7088f8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 常见的对称加密有：DES,AES等 
- 常见的非对称加密有：SSL,HTTPS,TLS,RSA。 

秘钥长度越长破解的难度越大。我在iOS持久化数据时用过AES，其他的没有用过。非对称加密中RSA是很有名，被应用的很广泛的数字证书。

## 数字签名和数字证书

看下面的一个场景：

A生成了一对“私钥-公钥”，然后把公钥给了B，B用公钥加密一段消息，发给A，A收到消息后用私钥解密，接着A用私钥加密一段文字给B，B收到后用公钥解密查看明文。如此反复。

上面是一个非对称加密聊天的具体场景，虽然经典，但是还是有一些问题。

```
1.A在收到B的信息的时候不能保证B发的信息是最原始的，即传输的内容有可能被篡改
2.黑客C获取了B的公钥，然后伪装成B和A进行通信

```

上面的情况都是可能出现的，现在这样，A为了保证数据不被改动，先对数据用hash函数生成一个摘要（digest）,然后用私钥对这个摘要进行加密，生成了“数字签名”（signature），B在收到这个消息时先用公钥对摘要进行解密得到摘要并且对数据内容也进行一次hash,比较这次的hash是否与之前解密得到的摘要一致，如果一致，说明数据没有被改动。我们把对内容数据进行hash后再加密生成的一段内容称为“**数字签名**”。

黑客C进行伪装交流的可能性也是有的，为了解决这个办法，A去找了一个权威的“证书中心”（certificate authority,简称CA），为公钥做验证。CA用自己的私钥对A的公钥和一些相关信息一起加密，生成了“**数字证书**”（Digital Certificate）。这样A在进行消息传递时也附带上数字证书，B在收到消息时用CA的公钥解开数字证书拿到真实的公钥，通过这样的方式验证身份。

阮一峰的[数字签名是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)很形象的说明了其中的意思。

## 苹果的证书
在Mac上的钥匙串访问app是专门用来管理证书的。iOS开发者在申请iOS开发证书时，需要通过keychain生成CSR文件（Certificate Signing Request），提交给苹果的`Apple Worldwide Developer Relations Certification Authority(WWDR)`证书认证中心进行签名，最后从苹果官网下载并安装使用。这个过程中还会产生一个私钥，证书和私钥在keychain中得位置如图：
![私钥](http://upload-images.jianshu.io/upload_images/1253942-0814c4c37eb06449.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. iOS系统原本就持有WWDR的公钥，系统首先会对证书内容通过指定的哈希算法计算得到一个信息摘要；
2. 然后使用WWDR的公钥对证书中包含的数字签名解密，从而得到经过WWDR的私钥加密过的信息摘要；
3. 最后对比两个信息摘要，如果内容相同就说明该证书可信。

整个过程如图所示：
![](http://upload-images.jianshu.io/upload_images/1253942-72e507584f9cae02?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在验证了证书是可信的以后，iOS系统就可以获取到证书中包含的开发者的公钥，并使用该公钥来判断代码签名的可用性了

在`objc.io`上面有篇[《Inside Code Signing》](https://www.objc.io/issues/17-security/inside-code-signing/)(中文翻译篇：[代码签名探析](http://objccn.io/issue-17-2/))上详细的讲述了一个已签名应用的组成和一些其他知识

# 自动打包
![image](http://upload-images.jianshu.io/upload_images/1253942-f218f61e611122fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

## 应用的构建过程和组成

想象一下平时的打包过程，在Xcode中选择对应的appid,bundleid,还要选择正确的配置文件（provisioning profile），然后点击`run`，我们看到Xcode上面有内容在不断的更新，正如猜想的，更新的内容实际就是app包编译的过程。

这个过程大概经过了：配置（编译器确定当前系统环境）-> 确定标准库和头文件的位置->确定依赖关系 ->头文件预编译(precompilation) -> 预处理(preprocessing) -> 编译(compilation) -> 连接(Linking) -> 打包 ，大致步骤是这些，但其中还有一些过程是没有讲的。

对于iOS的包来讲，在构建完成之后还会自动调用`codesign`命令进行签名，这个时候我们之前选择的bundleid啊，配置文件啊等等就排上用场了。经过签名后的应用是个相对来讲安全的应用，通过签名确保了包的来源合法，也能确保包的内容是否被修改过（理论知识上篇已讲过）。最后的包的本质实际上是一个Mach-O格式的二进制可执行文件（签名的数据就在这个二进制文件中）和一些资源文件。

### Mach-O可执行文件

Mach是一种操作系统内核。它的大致历史是：Mach内核被NeXT公司的NeXTSTEP操作系统使用，NeXT是乔布斯苹果被赶出苹果后创建的公司。1996年，乔布斯将NeXTSTEP带回苹果，成为了OS X的内核基础。在Mach上，一种可执行的文件格是就是Mach-O（Mach Object file format）。iOS是从OS X演变而来，所以同样支持Mach-O格式的可执行文件。

ipa包实际上就是一个zip压缩包，解压之后会有一个`Payload`文件夹，其中有个`XXX.app`这样的`.app`文件，它里面除了有个各种资源、图片等，还有个和包名相同的文件——这个就是二进制可执行文件。可以用`file`命令查看文件类型：
![image.png](https://upload-images.jianshu.io/upload_images/1253942-843929af7dad9f64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面看是支持arm7和arm64两种处理器架构的通用程序包，里面的格式是Mach-O。将可执行文件用Sublime打开，二进制开始部分如下：

[![wechat file](http://upload-images.jianshu.io/upload_images/1253942-03ba2dfe23a58f91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.vienta.me/img/autopacket/autopacket_6.png) 

开头的4个字节是cafebabe，这被称为“魔数”，反映文件的类型。查了下相关文章，OS X上还有如下几个标识：

- cafebabe:就是跨处理器架构的通用格式
- feedface和feedfacf则分别是某一处理器架构下的Mach-O格式
- 脚本的就很常见了，比如#!/bin/bash开头的shell脚本。

### 其他资源文件

解压后的包中除了可执行文件还有其他资源文件，图片啊，plist啊等等。苹果对安全确实重视，这些资源文件其实大多数也是需要被设置签名的，可以见到的是包中还有一个`_CodeSignature`文件夹，这个文件夹中的`CodeResources`文件中存储了被签名的程序包中所有需要被签名文件的签名。更详细的介绍参见[《代码签名探析》](http://objccn.io/issue-17-2/),从这些细节不难看出苹果对于安全的重视。

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

### 实战

自动打包是基础，打包完之后可以根据母包重新签名生成相似的包，生成的包可以自动部署。只要攻克这三个点就能实现全自动化

为了讲的更清楚，新建了一个项目`PackageExample`(Demo已上传到[这里](https://github.com/Vienta/BlogArticle))，并且使用了CocoaPods（实验起见仅引用了AFNetworking），项目的证书是dev状态的。`PackageExample`项目在我机器上的路径和目录如下截图：

![image](http://upload-images.jianshu.io/upload_images/1253942-c5e38379402c1688.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

和`PackageExample`同目录的还有`PackageShell`，里面的`buildipa.sh`为编译脚本，由于最后的目标是要做成可以随意配置的，所以还有一个`PackageConfig`文件夹，里面有配置文件`packageExample.mobileprovision`和`packageExample.plist`，配置文件主要用来签名，plist文件的内容为可配置的，例如里面有app_Prefix、app_Name、app_ID等信息。`Package`文件夹为打的包的存放的地方。

完整的编译脚本如下：

```
#!/bin/sh

#从plist文件中读取ipa包名和配置文件名
profile_Name=`/usr/libexec/PlistBuddy -c "print profile_Name" ./PackageConfig/packageExample.plist`
ipa_Name=`/usr/libexec/PlistBuddy -c "print app_Name" ./PackageConfig/packageExample.plist`

#进入工程目录
cd ../PackageExample
echo "go to packageExample workspace path"

#报名时根据时间戳命名的，所以这里有用到
buildTime=$(date +%Y%m%d%H%M)

profile="${profile_Name}"

echo $profile $ipa_Name

#一下方法主要是创建打包的路径和最后导出的ipa的路径
if [ ! -d "../PackageShell/Package" ]; then
    mkdir ../PackageShell/Package
fi

if [ ! -d "../PackageShell/Package/ArchiveProduction" ]; then
    mkdir ../PackageShell/Package/ArchiveProduction
fi

if [ ! -d "../PackageShell/Package/ArchiveProduction/QA" ]; then
    mkdir ../PackageShell/Package/ArchiveProduction/QA
    echo "Create ArchiveProduction path"
fi

if [ ! -d "../PackageShell/Package/ipa" ]; then
    mkdir ../PackageShell/Package/ipa
fi

if [ ! -d "../PackageShell/Package/ipa/QA" ]; then
    mkdir ../PackageShell/Package/ipa/QA
    echo "Create ipa path"
fi

buildConfiguration="QA"

buildPath="../PackageShell/Package/ArchiveProduction/QA/${ipa_Name}_${buildTime}.xcarchive"
ipaName="../PackageShell/Package/ipa/QA/${ipa_Name}_${buildTime}.ipa"

#先进行clean操作,clean的目的是进行清理缓存
#项目是workspace，所以这里必须要对应，如果是project则是project，
xctool -workspace PackageExample.xcworkspace -scheme PackageExample -configuration ${buildConfiguration} clean

#打包的命令，包的格式为xxx.xcarchive
xctool -workspace PackageExample.xcworkspace -scheme PackageExample -configuration ${buildConfiguration} archive -archivePath ${buildPath}
#导出ipa包的命令，
xcodebuild -exportArchive -exportFormat IPA -archivePath ${buildPath} -exportPath ${ipaName} -exportProvisioningProfile "$profile"

```
### PlistBuddy
脚本的开头有`PlistBuddy`命令，它是Mac下一个用来读写plist文件的工具，在/usr/libexec/下。
`xxx.xcarchive`包目录结构如图：
```
├── Info.plist
├── Products
├── SCMBlueprint
├── SwiftSupport
└── dSYMs
```


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


