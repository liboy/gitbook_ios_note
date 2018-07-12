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

### 沙盒路径结构
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

### Plist不安全
#### 什么是Plist文件？
属性列表（Plist,Property List）是一种结构化的二进制格式文件，包含了内嵌键值对的可执行bundle的基本配置信息。Plist文件主要用于存储App的用户设置及配置信息，例如，游戏类App经常会在Plist文件中存储游戏等级和分数信息。一般来说，App会将存储用户数据的Plist文件保存在“[App home目录]/documents/”目录下。Plist文件可以是XML格式或二进制格式。

#### Plist文件有哪些安全风险？
Plist文件主要用于存储用户设置及App的配置信息，但App可能使用Plist文件存储明文的用户名、密码或其它一些个人敏感信息。而保存在Plist文件中的二进制格式文件数据则可以使用Plist文件编辑器（如plutil）进行查看或修改，即使在一个没有越狱的设备上，plist文件也可以通过工具iExplorer获取。对于以编码、未加密或弱加密形式存储的敏感信息就可能会导致敏感信息泄露了。


## 三、偏好设置（NSUserDefaults）
- NSUserDefaults 是一个单例对象,在整个应用程序的生命周期中都只有一个实例。
- NSUserDefaults适合存储轻量级的本地数据，支持的数据类型有：NSNumber （NSInteger、float、double），NSString，NSDate，NSArray，NSDictionary，BOOL，NSData。
- NSUserDefaults一般保存配置信息，比如用户名、密码、是否保存用户名和密码、是否离线下载等一些配置条件信息。

>注意：UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。出现以上问题，可以通过调用synchornize方法强制写入`[defaults synchornize]`

优点：
- 不需要关心文件名
- 快速进行键值对存储
- 直接存储基本数据类型

缺点：
- 不能存储自定义数据
- 取出的数据都是不可变的

### NSUserDefaults不安全
NSUserDefaults其实是plist文件中键值存储，并且最大的问题是存在与沙盒中，这就对安全性埋下了隐患。如果攻击者破解app，拿到了沙盒中的数据，就会造成数据泄漏，后果不堪设想。

当然，一般也不会有把密码直接使用NSUserDefaults存储的，都会进行加密、或者是多重加密后再进行NSUserDefaults存储。这么做其实是可行的，前提是加密算法不能泄漏。有个小问题就是，如果用户删掉app重装的话，之前所有存储的敏感信息都会消失。比如，一个用户误删了使用NSUserDefaults存储密码的app，当重新安装之后，由于以前是记住密码免登录，只因为自己操作不当，接下来要进入找回密码功能，重新修改密码才能再次使用app。这对用户来说是一种相当不友好的体验。


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

## 五、SQLite3
SQLite3是一款开源的嵌入式关系型数据库，可移植性好、易使用、内存开销小。
SQLite3是无类型的，意味着你可以保存任何类型的数据到任意表的任意字段中。

### 使用步骤：
- 指定数据库路径。
- 创建sqlite3对象并且打开数据库。
- 创建表。
- 对数据库操作，包括增删改查。
- 关闭数据库。

## 六、CoreData

- CoreData提供了一种`对象-关系映射`的功能，能将OC对象转化成数据，保存Sqlite中。
- CoreData的好处就是能够合理管理内存，避免sql语句的麻烦(不用写sql语句)。

### CoreData构成

- NSManagedObjectContext:被管理的数据上下文，主要作用：插入、查询、删除。
- NSManagedObjectModel:数据库所有的表结构和数据结构，包含各个实体的定义的信息。主要作用就是添加实体、实体属性，建立属性之间的关系。
- NSPersistentStoreCoordinator持久化存储助理对象，相当于数据库的连接器。主要作用就是设置存储的名字、位置、存储方式。
- NSFetchRequest相当于select语句。查询封装对象。
- NSEntityDescription实体结构对象，相当于表格结构。
后缀为xxx.xcdatamodeld文件,编译后为xxx.momd的文件。

### CoreData版本升级和数据库迁移
- app中使用了CoreData,并且在下一个版本中有实体变动，比如实体新增字段、修改字段等改动，
- 那么app在覆盖安装时就要进行数据库迁移，否则app就会crash。
- Xcode8之后系统CoreData类做了改动，使用起来更简单了，如果您是用Xcode8创建的工程实现版本升级和数据迁移，则直接修改实体，然后重新生成即可，非常简单。
- 之前版本具体步骤参见：http://www.cocoachina.com/ios/20180605/23656.html
## 七、KeyChainhttp://www.cocoachina.com/ios/20180605/23656.html
Keychain是iOS所提供的一种安全存储参数的方式，最常用来存储账号，密码，用户信息，银行卡资料等信息，Keychain会以加密的方式存储在设备中

### Keychain的特点

- 钥匙串(KeyChain)是苹果公司Mac OS中的密码管理系统。
- 一个钥匙串可以包含多种类型的数据：密码（包括网站，FTP服务器，SSH帐户，网络共享，无线网络，群组软件，加密磁盘镜像等），私钥，电子证书和加密笔记等。
- iOS的KeyChain服务提供了一种安全的保存私密信息（密码，序列号，证书等）的方式。每个iOS程序都有一个独立的KeyChain存储。从iOS 3.0开始，跨程序分享KeyChain变得可行。
- 当应用程序被删除后，保存到KeyChain里面的数据不会被删除，所以KeyChain是保存到沙盒范围以外的地方。
- KeyChain的所有数据也都是以`key-value`的形式存储的，这和NSDictionary的存储方式一样。
- 相比于NSUserDefaults来说，KeyChain保存更为安全，而且KeyChain里面保存的数据不会因为app删除而丢失。
- 数据并不是放在App的Sanbox，即使删除了App，资料依然保存在keychain中，如果重新安装了App，还可以从keychain中获取数据
- keychain的数据可以用group的方式，让程序可以在App间共享，不过需要相同的TeamID

### Keychain的使用
- 首先导入Security.framework 框架
- Keychain的API提供以下几个函数来操作Keychain
    - SecItemAdd 添加一个keychain item
    - SecItemUpdate 修改一个keychain item
    - SecItemCopyMatching 搜索一个keychain item
    - SecItemDelete 删除一个keychain item

### KeychainItemWrapper的用法
[KeychainItemWrapper](https://github.com/baptistefetet/KeychainItemWrapper)是苹果封装Keychain API的类，封装了操作KeyChain的基本操作

```objectivec

// 初始化一个保存用户帐号的KeychainItemWrapper 
KeychainItemWrapper *wrapper = [[KeychainItemWrapper alloc] initWithIdentifier:@"Your Apple ID" accessGroup:@"YOUR_APP_ID.com.yourcompany.AppIdentifier"];
//参数forKey的值应该是Security.framework里头文件SecItem.h里定义好的key，用其他字符串做key程序会崩溃
//保存帐号
[wrapper setObject:@"<帐号>" forKey:(id)kSecAttrAccount];  
//保存密码
[wrapper setObject:@"<帐号密码>" forKey:(id)kSecValueData];      
//从keychain里取出帐号密码
NSString *password = [wrapper objectForKey:(id)kSecValueData];
//清空设置
[wrapper resetKeychainItem];


```

网上也有许多大神封装好的框架，我这里介绍两个点星上千的框架
SFHFKeychainUtils
SAMKeychain

### SFHFKeychainUtils
[SFHFKeychainUtils](https://github.com/ldandersen/scifihifi-iphone/tree/master/security)是另外一个第三方库，这个类比KeychainItemWrapper要简单很多，提供了更简单的方法保存密码到KeyChain

- SFHFKeychainUtils这个库是mrc，导入后可能会因为mrc会报错。

- SFHFKeychainUtils就3个方法：
```
//获取密码密码
+(NSString *) getPasswordForUsername: (NSString *) username andServiceName: (NSString *) serviceName error: (NSError **) error;
//存储密码
+(BOOL) storeUsername: (NSString *) username andPassword: (NSString *) password forServiceName: (NSString *) serviceName updateExisting: (BOOL) updateExisting error: (NSError **) error;
//删除密码
+(BOOL) deleteItemForUsername: (NSString *) username andServiceName: (NSString *) serviceName error: (NSError **) error;
```

- 参数说明
    - username：因为KeyChain保存也是以键值对存在，所以这个可以看作key，根据key取value.
    - forServiceName :这个就是组的名字，可以理解为KeyChain保存是分组保存。一般要唯一哦，命名可以使用YOUR_APP_ID.com.yourcompany.AppIdentifier。
    
- 如果两个应用的username、serviceName参数一样，那么这两个app会共用KeyChain里面的数据，也就是可以共享密码。

- KeyChain还有一个用途，就是替代UDID。UDID已经被废除了，所以只能用UUID代替，所以我们可以把UUID用KeyChain保存。
```objectivec
//创建一个uuid
NSString *uuidString = [self uuidString];
//31C75924-1D2E-4AF0-9C67-96D6929B1BD3
[SFHFKeychainUtils storeUsername:kKeyChainKey andPassword:uuidString forServiceName:kKeyChainGroupKey updateExisting:NO error:nil];
-(NSString *)uuidString
{
    //创建一个uuid
    CFUUIDRef uuidRef = CFUUIDCreate(kCFAllocatorDefault);
    CFStringRef stringRef = CFUUIDCreateString(kCFAllocatorDefault, uuidRef);
    
    NSString *uuidString = (__bridge NSString *)(stringRef);
    
    CFRelease(uuidRef);
    
    return uuidString;
}
```




