# SDWebImage

## 简介
SDWebImage 提供了 UIImageView、UIButton 、MKAnnotationView 的图片下载分类，只要一行代码就可以实现图片异步下载和缓存功能。

## 特性
- 显示网络图片，以及缓存管理
- 异步下载图片
- 异步缓存（内存+磁盘），并且自动管理缓存有效性
- 后台图片解压缩
- 同一个 URL 不会重复下载
- 自动识别无效 URL，不会反复重试
- 不阻塞主线程
- 高性能
- 使用 GCD 和 ARC
- 支持多种图片格式（包括 WebP 格式）
- 支持动图（GIF）
    - 4.0 之前的动图效果并不是太好
    - 4.0 以后基于 FLAnimatedImage加载动图