## 重新签名和授权机制

快捷查看系统中能用来对代码签名的证书
```
$security find-identity -v -p codesigning  
```
设置签名
```
$ codesign -s 'iPhone Developer: Thomas Kollbach (7TPNXN7G6K)' Example.app 
```
重新设置签名，你必须带上 -f 参数，有了这个参数，codesign 会用你选择的签名替换掉已经存在的那一个：
```
$ codesign -f -s 'iPhone Developer: Thomas Kollbach (7TPNXN7G6K)' Example.app  重新签名
```
列出一些有关 Example.app的签名信息
```
$ codesign -vv -d Example.app  
```
验证签名是否完好，若无任何输出则说明签名完好
```
$ codesign --verify Example.app  
```
### 授权文件（entitlements）
授权机制决定了哪些系统资源在什么情况下允许被一个应用使用，即沙盒的配置列表。授权机制也是按照 `plist` 文件格式来列出的，Xcode 会将这个文件作为 `–entitlements` 参数的内容传给 codesign ，这个文件内部格式如下：

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

- 选择 Xcode 的 `Capabilities` 选项，会自动生成一个 `.entitlements` 文件。
- 构建整个应用时，会提交给 `codesign` 作为应用所需要拥有哪些授权的参考。
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

### 重新签名的实例

设计思路如下图：
![image](http://upload-images.jianshu.io/upload_images/1253942-64d44600afabaeb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实例的`Resign-ipa`文件夹目录结构如下图：
[![image](http://upload-images.jianshu.io/upload_images/1253942-565b2e08ff4e0d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.vienta.me/img/autopacket/autopacket_10.png) 

`templates`文件夹中存放的是和授权文件相关的配置文件，`build`文件夹主要存放最后被重新签名的包，`module`目录下存放重签名的配置文件、资源文件和描述文件等。这里的思路是遍历`module`文件夹下的内容，然后根据内容重新签名母包，将得到的包放到`build`，例子中是两个，实际如果版本非常多，只需要向`module`增加文件夹及配置内容即可。

`resign.sh`为重签名的脚本，内容也不算多：

```bash
#!/bin/bash

for file2 in `ls -a ./module`
do
    if [ x"$file2" != x"." -a x"$file2" != x".." -a x"$file2" != x".DS_Store" ]; then
        echo $file2

        #Conf file
        CONF=./module/$file2/resign.conf

        echo $CONF

        #Datetime 
        NOW=$(date +"%Y%m%d_%s")

        #Load config
        if [ -f ${CONF} ]; then
            . ${CONF}
        fi

        #Temp
        TEMP="temp"
        if [ -e ${TEMP} ]; then
            echo "ERROR: temp already exists"
            exit 1
        fi

        #Check app ID
        if [ -z ${APP_ID} ]; then
            echo "ERROR: missing APP_ID"
            exit 1
        fi

        echo ${APP_ID}

        #Create build dir 
        if [[ ! -d ${BUILD_PATH} ]]; then
            mkdir ${BUILD_PATH}
        fi

        #Copy mother package
        if [[ ! -f  "../Package/ipa/QA/packageExample.ipa" ]]; then
            echo "mother package not exists"
            exit 1
        fi
        cp ../Package/ipa/QA/packageExample.ipa ./module/$file2${ASSETS_PATH}/packageExample.ipa

        #Unzip the mother ipa
        echo "Unzip ipa"
        unzip -q ./module/$file2${ASSETS_PATH}${IPA_NAME}.ipa -d ${TEMP}

        #Remove old Codesignature
        echo "Remove old CodeSignature"
        rm -r "${TEMP}/Payload/${APP_NAME}.app/_CodeSignature" "${TEMP}/Payload/${APP_NAME}.app/CodeResources" 2> /dev/null | true

        #Replace embedded mobil provisioning profile
        echo "Replace embedded mobile provisioning profile"
        cp "./module/$file2${ASSETS_PATH}${PROFILE_NAME}.mobileprovision" "${TEMP}/Payload/${APP_NAME}.app/embedded.mobileprovision"

        #Change icon
        echo "Change icon"
        cp "./module/$file2${ASSETS_PATH}/icon_120.png" "${TEMP}/Payload/${APP_NAME}.app/AppIcon60x60@2x.png"
        cp "./module/$file2${ASSETS_PATH}/icon_180.png" "${TEMP}/Payload/${APP_NAME}.app/AppIcon60x60@3x.png"

        #Change Bundleversion
        if [[ ! -z ${APP_BUNDLE_VERSION} ]]; then
            /usr/libexec/PlistBuddy -c "Set CFBundleVersion ${APP_BUNDLE_VERSION}" ${TEMP}/Payload/${APP_NAME}.app/Info.plist
        fi

        #Change CFBundleShortVersionString
        if [[ ! -z ${APP_BUNDLE_SHORT_VERSION_STRING} ]]; then
            /usr/libexec/PlistBuddy -c "Set CFBundleShortVersionString ${APP_BUNDLE_SHORT_VERSION_STRING}" ${TEMP}/Payload/${APP_NAME}.app/Info.plist
        fi

        #Change Bundleidentifier
        /usr/libexec/PlistBuddy -c "Set CFBundleIdentifier ${APP_ID}" ${TEMP}/Payload/${APP_NAME}.app/Info.plist

        #Create entitlements from template
        ENTITLEMENTS=$(<./templates/entitlements.template)
        ENTITLEMENTS=${ENTITLEMENTS//#APP_ID#/$APP_ID}
        ENTITLEMENTS=${ENTITLEMENTS//#APP_PREFIX#/$APP_PREFIX}
        echo ${ENTITLEMENTS} > ${TEMP}/entitlements.temp

        #Re-sign
        #这里注意命令参数的不同
        #/usr/bin/codesign -f -s "${CERTIFICATE_TYPE}: ${CERTIFICATE_NAME}" --identifier "${APP_ID}" --entitlements "${TEMP}/entitlements.temp" --resource-rules "${TEMP}/Payload/${APP_NAME}.app/ResourceRules.plist" "${TEMP}/Payload/${APP_NAME}.app"
        /usr/bin/codesign -f -s "${CERTIFICATE_TYPE}: ${CERTIFICATE_NAME}" --identifier "${APP_ID}" --entitlements "${TEMP}/entitlements.temp" "${TEMP}/Payload/${APP_NAME}.app"

         #Remove copyed mother package
        echo "Remove mother package"
        rm -rf ./module/$file2${ASSETS_PATH}packageExample.ipa

        #Re-package
        echo "Re-package"
        cd ${TEMP}
        zip -qr "${IPA_NAME}_resigned_${NOW}.ipa" Payload
        mv ${IPA_NAME}_resigned_${NOW}.ipa ../${BUILD_PATH}/${IPA_NAME}_${file2}_${NOW}.ipa

        #Remove temp
        cd ../
        rm -rf ${TEMP}

    fi
done
exit 0

```

代码中已经做了注释，做简单解释：

1.  最外面对`module`文件夹for循环
2.  读取`conf`配置文件（这些配置文件均是自己配置，实际也可以是plist文件，也比较方便）
3.  把母包从外面拷贝进来
4.  对拷贝过来的包进行解压，移除`CodeResources`等，替换描述文件，替换icon，修改BundleId及版本信息，修改授权文件（授权文件也是需要注意的点，inhouse和develop也有一点点区别，但整体没变）
5.  然后就是`codesign`命令，代码注释也指出了 `--resource-rules`参数的问题，我原本找到的是带这个参数的，但是现在用不到。具体原因[《代码签名探析》](http://objccn.io/issue-17-2/)这里说是：“伴随 OS X 10.10 DP 5 和 10.9.5 版本的发布，苹果改变了代码签名的格式，也改变了有关资源的规则。如果你使用10.9.5或者更高版本的 codesign 工具，在 CodeResources 文件中会有4个不同区域，其中的 rules 和 files 是为老版本准备的，而 files2 和 rules2是为新的第二版的代码签名准备的。**最主要的区别是在新版本中你无法再将某些资源文件排除在代码签名之外，在过去你是可以的，只要在被设置签名的程序包中添加一个名为 ResourceRules.plist 的文件**，这个文件会规定哪些资源文件在检查代码签名是否完好时应该被忽略。但是**在新版本的代码签名中，这种做法不再有效**。**所有的代码文件和资源文件都必须设置签名，不再可以有例外。**” 。简单的说，资源文件现在也必须重新签名了。
6.  重新压缩包，并移动到`build`中。

整个流程走完后我们在`build`中得到了重新签名后的包，并且可以通过iTunes按照到设备上且能够正常打开，但是我们发现icon被换掉了，内部的bundleId、版本号信息等也被换了，于是乎，就这样轻轻松松的**“换皮了”**。我的手机上最后的截图如下：

[![image](http://upload-images.jianshu.io/upload_images/1253942-97b20b550a71ee49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.vienta.me/img/autopacket/autopacket_11.png) 

优势：
1.  节省劳动
2.  配置方便，例子中主要修改icon，其实还可以加一些配置文件来配置颜色啊字体啊文字啊等等
3.  对于邪恶点的公司，申请很多app账号，然后用这个方法，快速换皮，app内部的内容接口根据bundleid等信息来配置，里面放放广告，对主版本进行导流量等。

