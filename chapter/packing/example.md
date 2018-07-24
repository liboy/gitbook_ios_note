# iOS应用程序的重签名(打包)

## 重签名方式
- Build后的app文件
- Archive包
- IPA文件

下面都是针对使用开发者证书签名，企业证书简单一些。
一、Xcode生成的应用的重签名
下面这三个重签名的需求主要是由我们工作决定的。我们有加固功能需要产品配合测试，当产品用他们自己的Xcode打包后，发给我们安装测试，由于证书的不一致或者他们没有企业证书，我们的手机是无法安装的，而公司的内测平台可以帮忙企业证书重签名，但必须是IPA文件，无形中加大了产品的工作量，所以我们希望不管产品发给我们是app还是archive包还是ipa，我们都能直接安装，那么这就需要我们自己来做重签名的事情了。

重签名脚本命令

### app重签名
1. 有效的证书（可以在钥匙串中查找）

2. mobileprovision 配置描述文件
可以在xcode中找一个有效的，重命名为`embedded.mobileprovision`拷贝到app的目录里


>注意:
- app是自己Xcode生成的，那这个`mobileprovision`文件可以直接使用现成的;
- 其他人开发的，根据该app包里的info.plist文件的`Bundle identifier`以及`capacity`来生成对应的mobileprovision文件才行

3. 生成entitlements.plist文件
先通过`security`命令，从mobileprovision文件中生成一个完整的plist文件
```bash
security cms -D -i "embedded.mobileprovision" > entitlements_full.plist

```
得到 `Entitlements` 字段
```
/usr/libexec/PlistBuddy -x -c 'Print:Entitlements'  entitlements_full.plist > entitlements.plist
```

4. 签名
同时签名的时候，需要带上entitlements.plist文件
```
/usr/bin/codesign --continue -f -s "证书" --entitlements "entitlements文件"  "需要签名的app文件path"
```

要想成功前面，下面四个条件缺一不可
(1) 证书要正确 

如果前面过程中，出现证书错误问题，请参考:签名证书错误

(2) 配置描述(embedded.mobileprovision)要正确 

包括appid，app group等信息

(3) 里面的framework都要签名，比如appx, dylib, framework 

(4)授权机制(entitlements.plist)文件

## IPA的重签名
### 解压IPA
```
unzip -qo "$SOURCE_IPA" -d "$TEMP_DIR"
-o:不提示的情况下覆盖文件；
-d:指明将文件解压缩的目录；
```
2. 删除旧的代码签名
rm -rf Payload/appName.app/_CodeSignature

这一步也可以忽略，在codesign的时候指定"-f"参数，强制替换就的签名
3. 更换证书

cp newEmbedded.mobileprovision Payload/appName.app/embedded.mobileprovision

4. 生成entitlements.plist文件
同上(App的重签名)

5. 签名

同上(App的重签名)

6. 重新打包，生成新的ipa


zip -r New_ appName.ipa Payload