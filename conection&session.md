# NSURLSession与NSURLConnection区别

## 1. 使用现状

- `NSURLSession`是`NSURLConnection` 的替代者，2013年随ios7一起发布，是对NSURLConnection进行了重构优化后的新的网络访问接口。
- `NSURLConnection`从`iOS9.0`之后发送请求方法、初始化网络连接方法也被设置为过期，系统不再推荐使用。
 

## 2. 普通任务和上传

`NSURLSession`针对下载、上传等复杂的网络操作提供了专门的解决方案，针对普通、上传和下载分别对应三种不同的网络请求任务：`NSURLSessionDataTask``NSURLSessionUploadTask``NSURLSessionDownloadTask`创建的task都是挂起状态，需要`resume`才能执行。 

- 当服务器返回的数据较小时，普通任务的操作步骤没有区别。 
- NSURLSession与NSURLConnection上传任务一样，都需要设置POST请求的请求体进行上传。
 

##3. 下载任务方式
- NSURLConnection下载文件时，先将整个文件下载到内存，然后再写入沙盒，如果文件比较大，就会出现内存暴涨的情况。
- 使用NSURLSessionDownloadTask下载文件，会默认下载到沙盒中的`temp`文件夹中，不会出现内存暴涨的情况，但在下载完成后会将`temp`中的临时文件删除，需要在初始化任务方法时，在completionHandler回调中增加保存文件的代码。 以下代码是实例化网络下载任务时将下载的文件保存到沙盒的caches文件夹中:

```objectivec
[NSURLSessionDownloadTask [NSURLSessionDownloadTask *task = [session downloadTaskWithURL:[NSURL URLWithString:@"http://192.168.1.17/xxxx.zip"] completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {
// 获取沙盒的caches路径 
NSString *path = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject]stringByAppendingPathComponent:@"xxx.dmg"];
// 生成URL路径
NSURL *url = [NSURL fileURLWithPath:path]; 
// 将文件保存到指定文件目录下 
[[NSFileManager defaultManager]moveItemAtURL:location toURL: url error:nil]; 
}] resume];
```
 

## 4、请求方法的控制

- NSURLConnection实例化对象开始，默认请求就发送（同步发送），不需要调用`start`方法。而`cancel`可以停止请求的发送，停止后不能继续访问，需要创建新的请求。 
- NSURLSession有三个控制方法：`取消cancel`、`暂停suspend`、`继续resume`，暂停后可以通过继续恢复当前的请求任务。
 

## 5、断点续传

使用NSURLSession进行断点下载更加便捷

- NSURLConnection进行断点下载，通过设置访问请求的`HTTPHeaderField`的`Range`属性，开启运行循环，`NSURLConnection`的代理方法作为运行循环的事件源，接收到下载数据时代理方法就会持续调用，并使用`NSOutputStream`管道流进行数据保存。 

- NSURLSession进行断点下载，当暂停下载任务后，如果 downloadTask （下载任务）为非空，调用 
```objectivec
cancelByProducingResumeData:(void (^)(NSData *resumeData))completionHandler
``` 
这个方法，这个方法接收一个参数，完成处理代码块，这个代码块有一个 NSData 参数 resumeData，如果 resumeData 非空，我们就保存这个对象到视图控制器的 resumeData 属性中。在点击再次下载时，通过调用 
```objectivec
[[self.session downloadTaskWithResumeData:self.resumeData] resume]
```
方法进行继续下载操作。 
 

## 6. 配置信息

- `NSURLSession`的构造方法`sessionWithConfiguration:delegate:delegateQueue`中有一个 `NSURLSessionConfiguration`类的参数可以设置配置信息，其决定了cookie，安全和高速缓存策略，最大主机连接数，资源管理，网络超时等配置。

- `NSURLConnection`依赖于一个全局的配置对象不能进行这个配置，缺乏灵活性而言，NSURLSession 有很大的改进了。


**NSURLSessionConfiguration**

- `defaultSessionConfiguration` 使用全局的cache，cookie,使用硬盘来缓存数据。
- `ephemeralSessionConfiguration` 临时session配置，与默认配置相比，这个配置不会将缓存、cookie等存在本地，只会存在内存里，所以当程序退出时，所有的数据都会消失
- `backgroundSessionConfiguration` 后台session配置，与默认配置类似，不同的是会在后台开启另一个线程来处理网络数据。