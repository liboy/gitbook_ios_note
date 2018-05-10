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
UIView+WebCacheOperation    用来记录图片加载的 operation，方便需要时取消和移除图片加载的 operation  
UIImageView+WebCache    集成 SDWebImageManager 的图片下载和缓存功能到 UIImageView 的方法中，方便调用方的简单使用  
UIImageView+HighlightedWebCache    跟 UIImageView+WebCache 类似，也是包装了 SDWebImageManager，只不过是用于加载 highlighted 状态的图片  
UIButton+WebCache    跟 UIImageView+WebCache 类似，集成 SDWebImageManager 的图片下载和缓存功能到 UIButton 的方法中，方便调用方的简单使用  
MKAnnotationView+WebCache    跟 UIImageView+WebCache 类似  
NSData+ImageContentType    用于获取图片数据的格式（JPEG、PNG等）  
UIImage+GIF    用于加载 GIF 动图  
UIImage+MultiFormat    根据不同格式的二进制数据转成 UIImage 对象  
UIImage+WebP    用于解码并加载 WebP 图片  
4. 核心逻辑


