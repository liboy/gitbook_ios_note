# 数据存储

## 1. iOS数据存储的几种方式
- 沙盒
- Plist存储
- Preference（偏好设置）
- NSKeyedArchiver归档 / NSKeyedUnarchiver解档
- SQLite3的使用
- FMDB
- Core Data

## 应用沙盒
- 每个iOS应用都有自己的应用沙盒(应用沙盒就是文件系统目录)，与其他文件系统隔离。应用必须待在自己的沙盒里，其他应用不能访问该沙盒

## 沙盒路径结构
- Document: 适合存储重要的数据，iTunes同步应用时会同步该文件下的内容，（比如游戏中的存档）
- Library
    - Caches：适合存储体积大，不需要备份的非重要数据，iTunes不会同步该文件
    - Preferences: 通常保存应用的偏好设置, iTunes同步该应用时会同步此文件夹中的内容
- tmp:保存应用的临时文件，用完就删除，系统可能在应用没在运行时删除该目录下的文件，iTunes不会同步该文件

| 路径 | 保存数据特点 | iTunes是否同步 |
| --- | --- | --- |	
| Document | 适合存储重要的数据 |	同步 |
| Library/Caches | 体积大，不需要备份 | 不同步 |
| Library/Preferences |	应用的设置信息 | 同步 |
| tmp |	临时文件 | 不同步 |

### App Bundle 里面有什么

- Info.plist:此文件包含了应用程序的配置信息，系统依赖此文件以获取应用程序的相关信息
- 可执行文件:此文件包含应用程序的入口和通过静态连接到应用程序target的代码
- 资源文件:图片,声音文件一类的
- 其他:可以嵌入定制的数据资源

