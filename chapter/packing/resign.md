## 重新签名

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
- 列出有关 Example.app的签名信息
```
$ codesign -vv -d Example.app  
```
- 验证签名是否完好，若无任何输出则说明签名完好
```
$ codesign --verify Example.app  
```

### 授权文件（entitlements）
授权机制决定了哪些系统资源在什么情况下允许被一个应用使用，即沙盒的配置列表。授权机制也是按照 `plist` 文件格式来列出的，这个文件内部格式如下：

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

### 描述文件（provisioning file）
在整个代码签名和沙盒机制中有一个组成部分将签名，授权和沙盒联系了起来，那就是描述文件 (provisioning profiles)。

#### OS X中保存目录
Xcode 将从开发者中心下载的全部配置文件都放在了这里：
```
~/Library/MobileDevice/Provisioning Profiles
```
#### 文件格式
描述文件并不是一个普通的plist文件，它是一个根据密码讯息语法 (Cryptographic Message Syntax) 加密的文件。

以XML格式查看该文件的命令：
```
$ security cms -D -i example.mobileprovision
```

#### 文件内容：
- UUID
每一个配置文件都有它自己的 UUID 。Xcode 会用这个 UUID 来作为标识，记录你在 build settings 中选择了哪一个配置文件。

- ProvisionedDevices
记录所有可用于调试的设备ID。

- DeveloperCertificates
包含了可以为使用这个配置文件的应用签名的所有证书。所有的证书都是基于 Base64 编码符合 PEM (Privacy Enhanced Mail, RFC 1848) 格式的。

- Entitlements
有关前面讲到的配置文件的所有内容都会被保存在这里。

### [实战](https://github.com/Vienta/BlogArticle/tree/master/package)
设计思路如下图：
![image](http://upload-images.jianshu.io/upload_images/1253942-64d44600afabaeb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](/assets/packing1.png)


实例的`Resign-ipa`文件夹目录结构如下图：
[![image](http://upload-images.jianshu.io/upload_images/1253942-565b2e08ff4e0d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.vienta.me/img/autopacket/autopacket_10.png) 

- `templates`文件夹中存放的是和授权文件相关的配置文件，
- `build`文件夹主要存放最后被重新签名的包，
- `module`目录下存放重签名的配置文件、资源文件和描述文件等。这里的思路是遍历`module`文件夹下的内容，然后根据内容重新签名母包，将得到的包放到`build`，例子中是两个，实际如果版本非常多，只需要向`module`增加文件夹及配置内容即可。
- `resign.sh`为重签名的脚本。

优势：
1.  节省劳动
2.  配置方便，例子中主要修改icon，其实还可以加一些配置文件来配置颜色啊字体啊文字啊等等
3.  对于邪恶点的公司，申请很多app账号，然后用这个方法，快速换皮，app内部的内容接口根据bundleid等信息来配置，里面放放广告，对主版本进行导流量等。

