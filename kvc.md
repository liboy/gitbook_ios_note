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
## 内部原理
1. 检查是否存在对应`key`的`set`或`get`方法，存在，就调用并赋值
2. 查找与key相同名称带有下划线的成员变量`_key`，有，就赋值
3. 查找相同名称的属性 `key`，有，就赋值
4. 都没有，则调用 `valueForUndefinedKey:`或`setValue:forUndefinedKey:`，默认实现都是抛出异常，可根据需要重写

## 使用场景

