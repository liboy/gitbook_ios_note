# Mac常用命令

## `defaults`命令
访问和修改Mac 上一些系统的默认设置（access the Mac OS X user defaults system）

### 显示隐藏`DashBoard`仪表盘
```
#若需恢复，将YES替换为NO即可
defaults write com.apple.dashboard mcx-disabled -boolean YES
killall Dock
```

### 显示隐藏文件
```
#还原的时候将true换成false
defaults write com.apple.finder AppleShowAllFiles true
killall Finder
```

### 显示Safari调试菜单
```
defaults write com.apple.safari IncludeDebugMenu -bool YES
killall Safari
```

### 改变系统默认截图文件属性
```
defaults write com.apple.screencapture type (jpg, jpeg, png)
```

### 显示`Xcode`的`build`的所用时间
```
default write com.apple.dt.Xcode ShowBuildOperationDuration YES
```

查看所有执行过的Defaults命令包括 defaults write, defaults read, defaults delete
```
history |grep "defaults"
```
分类查看也可以，只查看执行过的defaults write命令：
```
history |grep "defaults write"
```
只查看执行过的defaults delete命令：
```
history |grep "defaults delete"
```
查看关于某一个程序的defaults命令：
如果我们想查找有关某一个程序或者进程（Finder、Dock、Safari等）的命令，那么我们可以通过更改下面命令的字符来实现，例如下面要查找影响过Finder的defaults命令：
```
history |grep "defaults write com.apple.finder"
```
大多数程序都可以通过这样的方式找到，苹果的程序一般都是使用‘com.apple.appname’ 这样的语句。


## `sips`命令
---

使用`sips`批量缩放图片大小
```
sips -s format jpeg -Z 250 someImage.PNG --out myImage.JPEG
```
把someImage.PNG转换为最长边为250的myImage.JPEG。//什么叫最长边，比如我有张图它的比例是750*460，那么如果我用大Z的话，转出来的结果应该是`250 * 153（460 * （250/750))`，这就是最大边。 但如果我

想硬改为 `250 * 250` 命令如下:
```
sips -s format jpeg -z 250 250 someImage.PNG --out myImage.JPEG
```

 

zip 命令

>>

最通俗的用法

zip -q -r -e -m -o [yourName].zip someThing

-q 表示不显示压缩进度状态

-r 表示子目录子文件全部压缩为zip  //这部比较重要，不然的话只有something这个文件夹被压缩，里面的没有被压缩进去

-e 表示你的压缩文件需要加密，终端会提示你输入密码的

// 还有种加密方法，这种是直接在命令行里做的，比如zip -r -P Password01! modudu.zip SomeDir, 就直接用Password01!来加密modudu.zip了。

-m 表示压缩完删除原文件

-o 表示设置所有被压缩文件的最后修改时间为当前压缩时间

 

当跨目录的时候是这么操作的

zip -q -r -e -m -o '\user\someone\someDir\someFile.zip' '\users\someDir'