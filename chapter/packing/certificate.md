# 苹果证书
![image](http://upload-images.jianshu.io/upload_images/1253942-5aa0f5f1990499b5.jpg)

## 加密
* 对称加密 `symmetric cryptography`：加密和解密用的是同一个秘钥
* 非对称加密 `asymmetric cryptography`：加密和解密是不同的钥匙

### 对称加密
客户端（Client）和服务器（Server）进行通讯，`Client`和`Server`约定好相同的一把秘钥，Client发送的明文通过这把秘钥进行加密，Server在收到这段加密后的密文后通过事先约定好的那边秘钥进行解密得到明文。

![对称加密](http://upload-images.jianshu.io/upload_images/1253942-05697d31c8455025.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 非对称加密
每个端都生成一个 `私钥-公钥` 对，私钥是自己保管，公钥可以随便分享，用公钥可以解开私钥，用私钥可以解开公钥。
例如服务端生成了一个 `私钥-公钥` 对，自己保留了一份私钥，把公钥给客户端，客户端对发送的消息通过公钥进行加密，服务端在收到这个公钥后用自己的私钥进行解密还原得到明文。

![非对称加密](http://upload-images.jianshu.io/upload_images/1253942-07b9e270cc7088f8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 常见的对称加密有：`DES`,`AES`等 
- 常见的非对称加密有：`SSL`,`HTTPS`,`TLS`,`RSA`。 
- 秘钥长度越长破解的难度越大。
- 非对称加密中RSA是很有名，被应用的很广泛的数字证书。


## 数字签名和数字证书
## 数字签名（digital signature）
对指定信息使用哈希算法，得到一个固定长度的信息摘要`digest`，然后再使用私钥（注意必须是私钥）对该摘要加密，就得到了数字签名。所谓的代码签名就是这个意思。


上面的情况都是可能出现的，现在这样，A为了保证数据不被改动，先对数据用`hash`函数生成一个摘要,然后用私钥对这个摘要进行加密，生成了`数字签名（signature）`，B在收到这个消息时先用公钥对摘要进行解密得到摘要并且对数据内容也进行一次hash,比较这次的hash是否与之前解密得到的摘要一致，如果一致，说明数据没有被改动。我们把对内容数据进行hash后再加密生成的一段内容称为“**数字签名**”。

黑客C进行伪装交流的可能性也是有的，为了解决这个办法，A去找了一个权威的“证书中心”（certificate authority,简称CA），为公钥做验证。CA用自己的私钥对A的公钥和一些相关信息一起加密，生成了“**数字证书**”（Digital Certificate）。这样A在进行消息传递时也附带上数字证书，B在收到消息时用CA的公钥解开数字证书拿到真实的公钥，通过这样的方式验证身份。

阮一峰的[数字签名是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)很形象的说明了其中的意思。


## 证书和密匙
- 作为一个 iOS 开发者，在你开发使用的机器上应该已经有一个证书，一个公钥，以及一个私钥。这些是代码签名机制的核心。像 `SSL` 一样，代码签名也依赖于采用 `X.509` 标准的公开密钥加密。
- 在Mac上的`钥匙串访问`是专门用来管理证书的。

### 证书生成
- 通过keychain生成`CSR文件（Certificate Signing Request）`，
- 提交给苹果的`Apple Worldwide Developer Relations Certification Authority(WWDR)`证书认证中心进行签名
- 从苹果官网下载并安装使用。
        
这个过程中还会产生一个私钥，证书和私钥在keychain中位置如图：
![私钥](http://upload-images.jianshu.io/upload_images/1253942-0814c4c37eb06449.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>注意：如果你要导出证书，例如为了备份（强烈建议进行），一定要记得展开证书那一条显示出私钥并将两行都选中。

### 证书组成
- 证书本身
        包含用户的公钥、用户个人信息、证书颁发机构信息、证书有效期等信息。
- 证书签名
        证书本身内容的使用哈希算法得到一个固定长度的信息摘要，然后使用自己的私钥对该信息摘要加密生成数字签名
        
### 证书使用
1. iOS系统原本就持有`WWDR的公钥`，系统首先会对证书内容通过指定的哈希算法计算得到一个`信息摘要`
2. 然后使用`WWDR的公钥`对证书中包含的数字签名解密，从而得到经过WWDR的私钥加密过的`信息摘要`
3. 最后对比两个信息摘要，如果内容相同就说明该证书可信。

整个过程如图所示：
![](http://upload-images.jianshu.io/upload_images/1253942-72e507584f9cae02?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在验证了证书是可信的以后，iOS系统就可以获取到证书中包含的开发者的公钥，并使用该公钥来判断代码签名的可用性了

### 证书存在的意义

- 通过证书使用过程可以看出，证书本身只是一个中间媒介，iOS系统对证书并不关心，它其实只想要证书中包含的开发者的公钥！！
- 不管是哪一个开发者对iOS的安全系统说，这个公钥就是我的，系统是都不相信的，即系统对开发者有着百分之百的不信任感。但是iOS安全系统对自家的WWDR是可信任的，苹果将WWDR的公钥内置在了iOS系统中。有了证书，iOS安全系统只需要通过WWDR的公钥就可以获取到任何一个开发者的可信任的公钥了，这就是证书存在的意义！！

### 公钥（public key）
- 公钥被包含在数字证书里，数字证书又被包含在`描述文件(Provisioning File)`中，描述文件在应用被安装的时候会被拷贝到iOS设备中。
- iOS安全系统通过证书就能够确定开发者身份，就能够通过从证书中获取到的公钥来验证开发者用该公钥对应的私钥签名后的代码、资源文件等有没有被更改破坏，最终确定应用能否合法的在iOS设备上合法运行。

### 私钥（private key）
- 每个证书（其实是公钥）都对应有一个私钥，
- 私钥会被用来对代码、资源文件等签名。只有开发证书和描述文件是没办法正常调试的，因为没有私钥根本无法签名

## 代码签名探析
在`objc.io`上面有篇[《Inside Code Signing》](https://www.objc.io/issues/17-security/inside-code-signing/)(中文翻译篇：[代码签名探析](http://objccn.io/issue-17-2/))上详细的讲述了一个已签名应用的组成和一些其他知识

### 签名相关命令

- 查看系统中能用来对代码签名的证书
```
$security find-identity -v -p codesigning  
```
- 设置签名
```
$ codesign -s 'iPhone Developer: Thomas Kollbach (7TPNXN7G6K)' Example.app 
```
- 重新设置签名
```
//-f 参数会用选择的签名替换掉已经存在的
$ codesign -f -s 'iPhone Developer: Thomas Kollbach (7TPNXN7G6K)' Example.app  重新签名
```
- 列出有关签名信息
```
$ codesign -vv -d Example.app  
```
- 验证签名是否完好，若无任何输出则说明签名完好
```
$ codesign --verify Example.app  
```

### 授权文件（entitlements）
授权机制决定了哪些系统资源在什么情况下允许被使用，即沙盒的配置列表。

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">  
<plist version="1.0">  
<dict>  
    <key>application-identifier</key>
    <string>7TPNXN7G6K.ch.kollba.example</string>
    <key>aps-environment</key>
    <string>development</string>
    <key>com.apple.developer.team-identifier</key> 
    <string>7TPNXN7G6K</string>
    <key>com.apple.developer.ubiquity-container-identifiers</key>
    <array>
            <string>7TPNXN7G6K.ch.kollba.example</string>
    </array>
    <key>com.apple.developer.ubiquity-kvstore-identifier</key>
    <string>7TPNXN7G6K.ch.kollba.example</string>
    <key>com.apple.security.application-groups</key>
    <array>
            <string>group.ch.kollba.example</string>
    </array>
    <key>get-task-allow</key>
    <true/>
</dict>  
</plist>  
```

- 选择 Xcode 的 `Capabilities` 选项，会自动生成一个 `.entitlements` 文件，作为 `–entitlements` 参数的内容传给 codesign 
- 授权信息必须都在开发者中心的 App ID 中启用，并且包含在配置文件中。
- 在构建应用时可以在 `Xcode build setting` 中的 `code signing entitlements` 中设置。

### 描述文件
在整个代码签名和沙盒机制中有一个组成部分将`签名`、`授权`和`沙盒`联系了起来，那就是`描述文件 (provisioning profiles)`。

#### OS X中保存目录
Xcode 配置文件路径：
```
~/Library/MobileDevice/Provisioning Profiles
```
#### 文件格式
- 描述文件并不是一个普通的plist文件，它是一个根据`密码讯息语法 (Cryptographic Message Syntax)`简称 `CMS` 加密的文件。
- 命令行工具 `security` 可以解码这个 `CMS` 格式，以`XML`格式查看描述文件的命令：
```
$ security cms -D -i example.mobileprovision
```

#### 文件内容
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>AppIDName</key>
	<string>hangxiangSchool</string>
	<key>ApplicationIdentifierPrefix</key>
	<array>
	<string>4PH3U52Z85</string>
	</array>
	<key>CreationDate</key>
	<date>2018-02-02T03:15:39Z</date>
	<key>Platform</key>
	<array>
		<string>iOS</string>
	</array>
	<key>DeveloperCertificates</key>
	<array>
		<data>MIIFuTCCBKGgAwIBA....</data>
	</array>
	<key>Entitlements</key>
	<dict>
		<key>keychain-access-groups</key>
		<array>
			<string>4PH3U52Z85.*</string>		
		</array>
		<key>get-task-allow</key>
		<true/>
		<key>application-identifier</key>
		<string>4PH3U52Z85.com.xxx.xxx</string>
		<key>com.apple.developer.team-identifier</key>
		<string>4PH3U52Z85</string>
		<key>aps-environment</key>
		<string>development</string>
	</dict>
	<key>ExpirationDate</key>
	<date>2019-02-02T03:15:39Z</date>
	<key>Name</key>
	<string>52017_development</string>
	<key>ProvisionedDevices</key>
	<array>
		<string>xxxxx</string>
		<string>xxxxx</string>

	</array>
	<key>TeamIdentifier</key>
	<array>
		<string>4PH3U52Z85</string>
	</array>
	<key>TeamName</key>
	<string>Guangdong hangxiang culture technology co. LTD.</string>
	<key>TimeToLive</key>
	<integer>365</integer>
	<key>UUID</key>
	<string>0c245721-2417-40a5-bdb8-12c4a17c44b6</string>
	<key>Version</key>
	<integer>1</integer>
</dict>
```
- UUID
每一个配置文件都有它自己的 UUID 。Xcode 会用这个 UUID 来作为标识，记录你在 build settings 中选择了哪一个配置文件。

- `ProvisionedDevices`
记录所有可用于调试的设备ID。

- `DeveloperCertificates`
	- 包含了可以为使用这个配置文件的应用签名的所有证书。
	- 证书都是基于 Base64 编码符合 `PEM (Privacy Enhanced Mail, RFC 1848)` 格式。

	- 查看一个证书的详细内容，将编码过的文件内容复制粘贴到一个文件中去
```
-----BEGIN CERTIFICATE-----
MIIFuTCCBKGgAwIBA....
-----END CERTIFICATE-----
```
OpenSSL 来处理 
```
openssl x509 -text -in file.pem
```
- `Entitlements`
有关前面讲到的授权文件的所有内容都会被保存在这里。

### OpenSSL
是一个安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。

- 使用最新的openssl命令，方便研究SSL协议、数字证书。

如果你的openssl是 `LibreSSL` 查看[Mac安装新版OpenSSL问题](https://www.jianshu.com/p/32f068922baf)
更新前
```
$ openssl version
LibreSSL 2.2.7

$ which openssl
/usr/bin/openssl
```
更新后
```
$ openssl version
OpenSSL 1.0.2j  26 Sep 2016

$ which openssl
/usr/local/bin/openssl
```
如果更新之后还是没有显示正确的openssl，是因为系统存在两个openssl，通过which openssl命令可以查看，当前终端执行的openssl是哪个路径下的。可通过设置系统环境变量PATH来优先执行执行哪个路径下的openssl。

```
echo 'export PATH="/usr/local/Cellar/openssl/1.0.2o_1/bin/:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```
或创建软连接
```
ln -s /usr/local/Cellar/openssl/1.0.2j/bin/openssl /usr/local/bin
```
注意：`/usr/local/Cellar/openssl/1.0.2o_1/bin/` 该路径请按照你实际情况来更改,通常是1.0.2o_1这个文件夹不同！

