# KVC

## KVC概述

- KVC `Key-value coding` 键值编码。它是一种可以通过`字符串的名字（key）`来访问类属性的机制。而不是通过调用Setter、Getter方法访问。
- 关键方法定义在 `NSKeyValueCodingProtocol`
- KVC支持类对象和内建基本数据类型。
- KVC是KVO、Core Data、CocoaBindings的技术基础，他们都是利用了OC的动态性。

## KVC使用

* 获取值

```
- valueForKey: 传入NSString属性的名字。
- valueForKeyPath: 属性的路径xx.xx
- valueForUndefinedKey: 默认实现是抛出异常，可重写这个函数
    做错误处理
```

* 修改值

```
- setValue:forKey:
- setValue:forKeyPath:
- setValue:forUnderfinedKey:
- setNilValueForKey: 对非类对象属性设置nil时调用，默认抛出异常。
```
 
## 实现分析
KVC通过`isa-swizzing`实现其内部查找定位。`isa`指针（is kind of 的意思）指向维护方法表的对象的类，每个类都有一张方法表，是一个hash表，
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
    ![](/assets/kvc.png)
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

### 操作集合

Apple对KVC的valueForKey:方法作了一些特殊的实现，比如说NSArray和NSSet这样的容器类就实现了这些方法。所以可以用KVC很方便地操作集合

#### 用KVC实现高阶消息传递

当对容器类使用KVC时，valueForKey:将会被传递给容器中的每一个对象，而不是容器本身进行操作。结果会被添加进返回的容器中，这样，开发者可以很方便的操作集合来返回另一个集合。

```objectivec
NSArray* arrStr = @[@"english",@"franch",@"chinese"];
NSArray* arrCapStr = [arrStr valueForKey:@"capitalizedString"];
for (NSString* str  in arrCapStr) {
    NSLog(@"%@",str);
}
NSArray* arrCapStrLength = [arrStr valueForKeyPath:@"capitalizedString.length"];
for (NSNumber* length  in arrCapStrLength) {
    NSLog(@"%ld",(long)length.integerValue);
}
```


#### 用KVC中的函数操作集合
- 简单集合运算符@avg， @count ， @max ， @min ，@sum5种
```objectivec
//属性相加
NSString *sum= [persons valueForKeyPath:@"Person.@sum.age"];
NSLog(@"sum = %@",sum);
//数量
NSString *count= [persons valueForKeyPath:@"Person.@count.age"];
NSLog(@"count = %@",count);
//最大值
NSString *max= [persons valueForKeyPath:@"Person.@max.age"];
NSLog(@"max = %@",max);
//最小值
NSString *min= [persons valueForKeyPath:@"Person.@min.age"];
NSLog(@"min = %@",min);
//平均值
NSString *avg= [persons valueForKeyPath:@"Person.@avg.age"];
NSLog(@"avg = %@",avg);
```
- 对象运算符
比集合运算符稍微复杂，能以数组的方式返回指定的内容，一共有两种：
```objectivec
@distinctUnionOfObjects
@unionOfObjects
```
它们的返回值都是NSArray，区别是前者返回的元素都是唯一的，是去重以后的结果；后者返回的元素是全集。
用法如下：
```objectivec
NSLog(@"distinctUnionOfObjects");
NSArray* arrDistinct = [arrBooks valueForKeyPath:@"@distinctUnionOfObjects.price"];
for (NSNumber *price in arrDistinct) {
    NSLog(@"%f",price.floatValue);
}
NSLog(@"unionOfObjects");
NSArray* arrUnion = [arrBooks valueForKeyPath:@"@unionOfObjects.price"];
for (NSNumber *price in arrUnion) {
    NSLog(@"%f",price.floatValue);
}
```        

- Array和Set操作符
这种情况更复杂了，说的是集合中包含集合的情况，我们执行了如下的一段代码：
```objectivec
@distinctUnionOfArrays
@unionOfArrays
@distinctUnionOfSets
```
    - `@distinctUnionOfArrays`：该操作会返回一个数组，这个数组包含不同的对象，不同的对象是在从关键路径到操作器右边的被指定的属性里
    - `@unionOfArrays` 该操作会返回一个数组，这个数组包含的对象是在从关键路径到操作器右边的被指定的属性里和@distinctUnionOfArrays不一样，重复的对象不会被移除
    - `@distinctUnionOfSets` 和`@distinctUnionOfArrays`类似。因为Set本身就不支持重复。
