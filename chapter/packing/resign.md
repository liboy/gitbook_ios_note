# 重新签名

## 几种方式
### 1. 工具iReSign
https://www.jianshu.com/p/6fe9eb030922
1. 就是ipa的路径，点击浏览就能添加
2. 就是新的配置文件的路径
3. 是entitlement.plist的路径
4. 重新修改成的APPID ，后面要打对号（如果需要修改APPID的话，同样开发者账号中要新增或者修改成新的APPID）
5. 就是所对应的证书，双击导入到钥匙串，这里自然会显示，如果不显示，关掉iReSign再打开就可以了

### 2. 终端命令行

1.解压ipa包
```
unzip youApp.ipa
```
2.删除解压后包内的_CodeSignature文件夹，解除之前的签名
```bash
rm -rf Payload/YourApp.app/_CodeSignature
```
3.替换解压包内的配置文件 
```bash
cp ~/Downloads/AdHoc.mobileprovision Payload/YouApp.app/embedded.mobileprovision
```

4.签名 codesign -f -s “证书名字” 目标文件
```bash
codesign -f -s "iPhone Developer: shize zhong (EMDFFQCRZQ)" /Users/hfios/Desktop/Payload/YouApp.app
```

5.压缩成ipa
```
zip -r new.ipa Payload
```

### 3. sign脚本

安装好brew，先用brew安装ruby，然后用gem安装sigh。（brew去网上搜一下）
```bash
brew install ruby
sudo gem install sigh
```
使用就非常简单了：
1、输入sigh resign，回车
2、把要签名的ipa文件拖到窗口上，回车
3、填写用来签名的证书，回车
4、把embedded.mobileprovision文件拖到窗口上，回车
5、好了，resign脚本会自动更改bundel id，签名并重新打包。

```bash
security cms -D -i "embedded.mobileprovision" > t_entitlements_full.plist
/usr/libexec/PlistBuddy -x -c 'Print:Entitlements' t_entitlements_full.plist > entitlements.plist
Entitlements=entitlements.plist

codesign -f -s "$tcertificationname" --entitlements $Entitlements ${tapppackagepath}
```

## 实战

### 实例1[`AutoArchive`](https://github.com/yang152412/AutoArchive)
xcode 8后，原来用企业证书 wildcard 打包，会因为没有 APNS 报错，所以只能指定 bundleIdentifier，但是指定之后，就不能和正常上线的 bundleIdentifier 一致了，每次打包就要重新设置一下 bundle identifier。解决方案 1、用两个 target，分开设置。2、makefile 打包的时候分别 设置 企业证书 和 上线证书。本文采用 第二种方案



设计思路如下图：

![](/assets/packing/resign2.png)
![](/assets/packing/resign1.png)


### 实例2[`Resign-ipa`](https://github.com/Vienta/BlogArticle/tree/master/package)
文件夹目录结构如下图：
[![image](http://upload-images.jianshu.io/upload_images/1253942-565b2e08ff4e0d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.vienta.me/img/autopacket/autopacket_10.png) 

- `templates`文件夹中存放的是和授权文件相关的配置文件，
- `build`文件夹主要存放最后被重新签名的包，
- `module`目录下存放重签名的配置文件、资源文件和描述文件等。这里的思路是遍历`module`文件夹下的内容，然后根据内容重新签名母包，将得到的包放到`build`，例子中是两个，实际如果版本非常多，只需要向`module`增加文件夹及配置内容即可。
- `resign.sh`为重签名的脚本。

优势：
1.  节省劳动
2.  配置方便，例子中主要修改icon，其实还可以加一些配置文件来配置颜色啊字体啊文字啊等等
3.  对于邪恶点的公司，申请很多app账号，然后用这个方法，快速换皮，app内部的内容接口根据bundleid等信息来配置，里面放放广告，对主版本进行导流量等。

