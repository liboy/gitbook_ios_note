# SDWebImage

## 简介

SDWebImage 提供了 UIImageView、UIButton 、MKAnnotationView 的图片下载分类，只要一行代码就可以实现图片异步下载和缓存功能。

## 特性

* 显示网络图片，以及缓存管理
* 异步下载图片
* 异步缓存（内存+磁盘），并且自动管理缓存有效性
* 后台图片解压缩
* 同一个 URL 不会重复下载
* 自动识别无效 URL，不会反复重试
* 不阻塞主线程
* 高性能
* 使用 GCD 和 ARC
* 支持多种图片格式（包括 WebP 格式）
* 支持动图（GIF）
  * 4.0 之前的动图效果并不是太好
  * 4.0 以后基于 FLAnimatedImage加载动图

## 实现原理

1. 架构图（UML 类图）
![](/assets/sd_1.png)
2. 流程图（方法调用顺序图）
![](/assets/sd_2.png)

### 目录结构

* Downloader
  * SDWebImageDownloader
  * SDWebImageDownloaderOperation
* Cache
  * SDImageCache
* Utils
  * SDWebImageManager
  * SDWebImageDecoder
  * SDWebImagePrefetcher
* Categories
  * UIView+WebCacheOperation
  * UIImageView+WebCache
  * UIImageView+HighlightedWebCache
  * UIButton+WebCache
  * MKAnnotationView+WebCache
  * NSData+ImageContentType
  * UIImage+GIF
  * UIImage+MultiFormat
  * UIImage+WebP
* Other
  * SDWebImageOperation（协议）
  * SDWebImageCompat（宏定义、常量、通用函数）

| 类名 | 功能  |
| :--- | :--- |
| SDWebImageDownloader | 是专门用来下载图片和优化图片加载的，跟缓存没有关系 | 
| SDWebImageDownloaderOperation | 继承于 NSOperation，用来处理下载任务的 |
| SDImageCache | 用来处理内存缓存和磁盘缓存（可选\)的，其中磁盘缓存是异步进行的，因此不会阻塞主线程 | 
|SDWebImageManager|作为 UIImageView+WebCache 背后的默默付出者，主要功能是将图片下载（SDWebImageDownloader）和图片缓存（SDImageCache）两个独立的功能组合起来  |
|SDWebImageDecoder  |  图片解码器，用于图片下载完成后进行解码|  
|SDWebImagePrefetcher  |  预下载图片，方便后续使用，图片下载的优先级低，其内部由 SDWebImageManager 来处理图片下载和缓存  |
|UIView+WebCacheOperation  |  用来记录图片加载的 operation，方便需要时取消和移除图片加载的 operation | 
|UIImageView+WebCache  |  集成 SDWebImageManager 的图片下载和缓存功能到 UIImageView 的方法中，方便调用方的简单使用  
|UIImageView+HighlightedWebCache  |  跟 UIImageView+WebCache 类似，也是包装了 SDWebImageManager，只不过是用于加载 highlighted 状态的图片  |
|UIButton+WebCache  |  跟 UIImageView+WebCache 类似，集成 SDWebImageManager 的图片下载和缓存功能到 UIButton 的方法中，方便调用方的简单使用  |
|MKAnnotationView+WebCache  |  跟 UIImageView+WebCache 类似|  
|NSData+ImageContentType  |  用于获取图片数据的格式（JPEG、PNG等）  |
|UIImage+GIF |   用于加载 GIF 动图  |
|UIImage+MultiFormat |   根据不同格式的二进制数据转成 UIImage 对象  |
|UIImage+WebP  |  用于解码并加载 WebP 图片  |


## 工作流程

1. 入口 `setImageWithURL:placeholderImage:options:` 会先把 `placeholderImage` 显示，然后 `SDWebImageManager` 根据 URL 开始处理图片。

- 进入 `SDWebImageManager-downloadWithURL:delegate:options:userInfo:`交给 SDImageCache 从缓存查找图片是否已经下载 `queryDiskCacheForKey:delegate:userInfo:`。

- 先从内存图片缓存查找是否有图片，如果内存中已经有图片缓存，SDImageCacheDelegate 回调 `imageCache:didFindImage:forKey:userInfo:` 到 SDWebImageManager。

- `SDWebImageManagerDelegate` 回调 `webImageManager:didFinishWithImage:` 到 UIImageView+WebCache 等前端展示图片。

- 如果内存缓存中没有，生成 `NSInvocationOperation` 添加到队列开始从硬盘查找图片是否已经缓存。

- 根据 `URLKey` 在硬盘缓存目录下尝试读取图片文件。这一步是在 NSOperation 进行的操作，所以回主线程进行结果回调 notifyDelegate:。

- 如果从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate 回调 `imageCache:didFindImage:forKey:userInfo:`进而回调展示图片。

- 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 `imageCache:didNotFindImageForKey:userInfo:`。

- 共享或重新生成一个下载器 `SDWebImageDownloader` 开始下载图片。

- 图片下载由 NSURLConnection(3.8.0之后使用了NSURLSession)，实现相关 delegate 来判断图片下载中、下载完成和下载失败。

- `connection:didReceiveData:` 中利用 ImageIO 做了按图片下载进度加载效果。connectionDidFinishLoading: 数据下载完成后交给 `SDWebImageDecoder` 做图片解码处理。

- 图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。

- 在主线程 `notifyDelegateOnMainThreadWithInfo:` 宣告解码完成，`imageDecoder:didFinishDecodingImage:userInfo:` 回调给 SDWebImageDownloader。

- `imageDownloader:didFinishWithImage:` 回调给 SDWebImageManager 告知图片下载完成。

- 通知所有的 `downloadDelegates` 下载完成，回调给需要的地方展示图片。

- 将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 NSInvocationOperation 完成，避免拖慢主线程。

- SDImageCache 在初始化的时候会注册一些消息通知，在内存警告或退到后台的时候清理内存图片缓存，应用结束的时候清理过期图片。

- SDWebImagePrefetcher 可以预先下载图片，方便后续使用。



