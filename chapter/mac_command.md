# defaults 命令详解

defaults:该命令 访问和修改Mac 上一些系统的默认设置（access the Mac OS X user defaults system）
1 隐藏DashBoard
```bash
defaults write com.apple.dashboard mcx-disabled -boolean YES
killall Dock
```
DashBoard里面有很多小工具，可惜并不是对每个人有用，通过上述命令即可隐藏，若需恢复，将YES替换为NO即可。


2 显示隐藏文件
1
2
defaults write com.apple.finder AppleShowAllFiles true
killall Finder
还原的时候将true换成false即可。


3显示Safari调试菜单
1
2
defaults write com.apple.safari IncludeDebugMenu -bool YES
killall Safari
4 显示Xcode 每一次build的所用时间
1
2
default write com.apple.dt.Xcode ShowBuildOperationDuration YES

显示 

5 查看所有执行过的Defaults命令包括 defaults write, defaults read, defaults delete
history |grep "defaults"
复制代码
分类查看也可以，只查看执行过的defaults write命令：
history |grep "defaults write"
复制代码
只查看执行过的defaults delete命令：
history |grep "defaults delete"
复制代码
查看关于某一个程序的defaults命令：
如果我们想查找有关某一个程序或者进程（Finder、Dock、Safari等）的命令，那么我们可以通过更改下面命令的字符来实现，例如下面要查找影响过Finder的defaults命令：
history |grep "defaults write com.apple.finder"
复制代码
大多数程序都可以通过这样的方式找到，苹果的程序一般都是使用‘com.apple.appname’ 这样的语句。