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
- Document:适合存储重要的数据， iTunes同步应用时会同步该文件下的内容，（比如游戏中的存档）
- Library/Caches：适合存储体积大，不需要备份的非重要数据，iTunes不会同步该文件
- Library/Preferences: 通常保存应用的设置信息, iTunes同步该应用时会同步此文件夹中的内容
- tmp:保存应用的临时文件，用完就删除，系统可能在应用没在运行时删除该目录下的文件，iTunes不会同步该文件

| 路径 | 保存数据特点 | iTunes是否同步 |
| --- | --- | --- |	
| Document | 适合存储重要的数据 |	同步 |
| Library/Caches | 体积大，不需要备份 | 不同步 |
| Library/Preferences |	应用的设置信息 | 同步 |
| tmp |	临时文件 | 不同步 |

### 获取沙盒路径
3.1 获取 Document / Caches下的路径

NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
第一个常量NSDocumentDirectory表示正在查找沙盒Document目录的路径（如果参数为NSCachesDirectory则表示沙盒Cache目录）
第二个常量NSUserDomainMask表明我们希望将搜索限制在应用的沙盒内

NSString *documentFilePath = paths.firstObject;
因为每一个应用只有一个Documents目录，所以这里取第一个和最后一个数据都是一样的
3.2 获取 tmp下的路径

- (NSString *)getCacheFilePath
{
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    return paths.firstObject;
}

作者：哆啦_
链接：https://www.jianshu.com/p/ff89a6cdd927
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。