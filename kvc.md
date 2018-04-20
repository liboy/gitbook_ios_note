# KVC

## KVC概述

- KVC `Key-value coding` 键值编码。它是一种可以通过`字符串的名字（key）`来访问类属性的机制。而不是通过调用Setter、Getter方法访问。
- 关键方法定义在 NSKeyValueCodingProtocol
- KVC支持类对象和内建基本数据类型。

## KVC使用

* 获取值
    - valueForKey: 传入NSString属性的名字。
    - valueForKeyPath: 属性的路径，xx.xx
    - valueForUndefinedKey 默认实现是抛出异常，可重写这个函数做错误处理

* 修改值
    - setValue:forKey:
    - setValue:forKeyPath:
    - setValue:forUnderfinedKey:
    - setNilValueForKey: 对非类对象属性设置nil时调用，默认抛出异常。
    
## 实现分析
KVC通过`isa-swizzing`实现其内部查找定位。isa指针（is kind of 的意思）指向维护方法表的对象的类，每个类都有一张方法表，是一个hash表，
- 值是函数指针IMP
- 键key->SEL的名称

```objectivec
[site setValue:@"sitename" forKey:@"name"];

//会被编译器处理成
SEL sel = sel_get_uid(setValue:forKey);
IMP method = objc_msg_loopup(site->isa,sel);
method(site,sel,@"sitename",@"name");
```
KVC键值查找

- setValue:forKey:搜索方式

1、首先搜索setKey:方法。（key指成员变量名，首字母大写）

2、上面的setter方法没找到，如果类方法accessInstanceVariablesDirectly返回YES。那么按 _key，_isKey，key，iskey的顺序搜索成员名。（NSKeyValueCodingCatogery中实现的类方法，默认实现为返回YES）

3、如果没有找到成员变量，调用setValue:forUnderfinedKey:

- valueForKey:的搜索方式

1、首先按getKey，key，isKey的顺序查找getter方法，找到直接调用。如果是BOOL、int等内建值类型，会做NSNumber的转换。

2、上面的getter没找到，查找countOfKey、objectInKeyAtindex、KeyAtindexes格式的方法。如果countOfKey和另外两个方法中的一个找到，那么就会返回一个可以响应NSArray所有方法的代理集合的NSArray消息方法。

3、还没找到，查找countOfKey、enumeratorOfKey、memberOfKey格式的方法。如果这三个方法都找到，那么就返回一个可以响应NSSet所有方法的代理集合。
4、还是没找到，如果类方法accessInstanceVariablesDirectly返回YES。那么按 _key，_isKey，key，iskey的顺序搜索成员名。

5、再没找到，调用valueForUndefinedKey。