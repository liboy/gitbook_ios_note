# 数据存储

## 1. iOS数据存储的几种方式
- 沙盒
- Plist存储
- Preference（偏好设置）
- NSKeyedArchiver归档 / NSKeyedUnarchiver解档
- SQLite3的使用
- FMDB
- Core Data

## 一、应用沙盒
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

## 二、Plist存储
Plist文件的Type可以是字典NSDictionary或数组NSArray，也就是说可以把字典或数组直接写入到文件中。


## 三、偏好设置（NSUserDefaults）
- NSUserDefaults 是一个单例对象,在整个应用程序的生命周期中都只有一个实例。
- NSUserDefaults适合存储轻量级的本地数据，支持的数据类型有：NSNumbe （NSInteger、float、double），NSString，NSDate，NSArray，NSDictionary，BOOL，NSData。
- NSUserDefaults一般保存配置信息，比如用户名、密码、是否保存用户名和密码、是否离线下载等一些配置条件信息。

>注意：UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。出现以上问题，可以通过调用synchornize方法强制写入`[defaults synchornize]`

优点：
- 不需要关心文件名
- 快速进行键值对存储
- 直接存储基本数据类型

缺点：
- 不能存储自定义数据
- 取出的数据都是不可变的

## 四、NSKeyedArchiver归档（NSCoding）

- NSKeyedArchiver归档
和NSKeyedUnarchiver解档会在写入、读出数据之前进行序列化、反序列化，数据的安全性相对高一些。
- 如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，可以直接用NSKeyedArchiver进行归档和恢复
- 不是所有的对象都可以直接用这种方法进行归档，只有遵守了NSCoding协议的对象才可以
- NSCoding协议有2个方法：
    - encodeWithCoder:
每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用`encodeObject:forKey:`方法归档实例变量
    - initWithCoder:
每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用`decodeObject:forKey`方法解码实例变量

## 五、SQLite3的使用
SQLite3是一款开源的嵌入式关系型数据库，可移植性好、易使用、内存开销小
SQLite3是无类型的，意味着你可以保存任何类型的数据到任意表的任意字段中。

1、首先需要添加库文件libsqlite3.0.tbd
2、导入头文件#import <sqlite3.h>
3、打开数据库
4、创建表
5、对数据表进行增删改查操作
6、关闭数据库
SQLite3
