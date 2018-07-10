

推送和通知NSNotification 的区别 

1.NSNotification是系统内部发出的通知，一般用于内部事件的监听，或者状态的改变等、是不可见的（建议不要滥用NSNotification,因为是在主线程中执行，使用不当会发生线程阻塞）

2.本地通知与远程通知是可见的，主要用户告知用户或者发送一些App的内容更新，推送一些消息，让App知道App内部发生了什么事情。


2、远程推送通知：
远程推送服务APNS（Apple Push Notification Servers）
所有苹果设备在联网的情况下都会与苹果建立长连接
长连接：
1.只要联网就一直建立连接
2.长连接作用：1.时间校准2.系统升级3.查找我的iPhone等 
长连接的好处
1.数据传输速度快 
2.数据保持最新状态 
远程推送的基本过程
1.客户端app需要将用户的UUID和app的bundleID发送给apps服务器，进行注册，apps服务器将加密后的Device Token返回给app 
2.app获得Device Token后，上传到公司的服务器 
3.当需要推送通知时，公司的服务器会将推送内容和Device Token一起发给apns服务器 
4.apns 再将推送的内容推送给客户端 