## 重新签名

https://www.jianshu.com/p/6fe9eb030922

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

