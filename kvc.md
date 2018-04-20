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
KVC在iOS开发中是绝不可少的利器，这种基于运行时的编程方式极大地提高了灵活性，简化了代码，甚至实现很多难以想像的功能，KVC也是许多iOS开发黑魔法的基础。下面我来列举iOS开发中KVC的使用场景
- 动态地取值和设值
- 访问和修改私有变量
- Model和字典转换
- 修改一些控件的内部属性
    - 在xib/Storyboard中，也可以使用KVC，下面是在xib中使用KVC把图片边框设置成圆角
- 用KVC中的函数操作集合

KVC默认的实现方法有NSOject提供，这种方法及支持对象也支持简单数据类型。
### 访问变量的几种方式：
1. `public`型，通过 `->` 直接访问：
```objectivec
@interface Book : NSObject
{
    @public
    NSString *name;
}
Book *book=[[Bookalloc]init];
book->name=@"hello";
NSLog(@"val is %@",book->name);
```
2. 利用属性访问
3. 利用KVC,即使该属性是private也可以访问
```objectivec
@interface Book : NSObject
{
    @private
    NSString *name;
}
Book *book=[[Book alloc]init];
[book setValue:@"hello"forKey:@"name"];
NSLog(@"val is %@",[bookvalueForKey:@"name"]);
```

### KVC路径访问
```objectivec
[book valueForKeyPath:@"authorObj.name"]; 
author *authorObj=[[author alloc] init];
[authorObj setValue:@"niudun" forKey:@"name"];
[book setValue:authorObj forKey:@"authorObj"];
NSLog(@"the author of book is%@",[book valueForKeyPath:@"authorObj.name"]);
```
     
### 一对多

```objectivec
@interface Book : NSObject
{
    @private
    NSString *name;
    author *authorObj;
    NSArray *relativeBooks;
    
}        
//一对多
NSMutableArray *array=[NSMutableArray arrayWithCapacity:3];
for (int i=0; i<3; i++) {
            Book *bookObj=[[Book alloc] init];
            NSString *name=[NSString stringWithFormat:@"job_%d",i];
            [bookObj setValue:name forKey:@"name"];
            [array addObject:bookObj];
        }
        [book setValue:array forKey:@"relativeBooks"];
        NSArray *arr=[book valueForKeyPath:@"relativeBooks.name"];
        NSLog(@"arr is %@",arr);
```
第四、kvc支持简单的预算如max、min、sum，其中运算的字段必须是基本数据类型或NSNumber类型
第五、KVC对数值和结构体类型的支持
一套机制如果不支持数值和结构体型的数据，那么它的实用性就会大大折扣。幸运的是KVC中苹果对这方面的支持做的很好。KVC可以自动的将数值或结构体型的数据打包或解包成NSNumber或NSValue对象，以达到适配的目的。
举个例子，Person类有个个NSInteger类型的num属性
①修改值
我们通过KVC技术使用如下方式设置age属性的值： 
[_person setValue:[NSNumber numberWithInteger:5] forKey:@"num"];
我们赋给num的是一个NSNumber对象，KVC会自动的将NSNumber对象转换成NSInteger对象，然后再调用相应的访问器方法设置age的值。
②获取值
同样，以如下方式获取age属性值：
     [person valueForKey:@"num"];
这时，会以NSNumber的形式返回num的值。
第六、KVC实现原理的方法定义
在iOS中，通过KVC可以直接用字符串的名字(key)来访问类属性的机制。而不是通过调用Setter、Getter方法访问。
KVC是KVO、Core Data、CocoaBindings的技术基础，他们都是利用了OC的动态性。
NSKeyValueCodingprotocol
