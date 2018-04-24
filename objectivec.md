# Objective-C



## @property、@synthesize和@dynamic分别有什么作用？
```objectivec
@property = ivar(实例变量) + getter(取方法) + setter(存方法);
```
- ivar、getter、setter是编译器自动合成的，通过`@synthesize`指定，若不指定，默认为 `@synthesize var = _var`

- `@synthesize`：是如果你没有手动实现getter与setter方法，那么编译器就会自动为你加上这两个方法。

- `@dynamic`：告诉编译器，getter与setter方法由用户自己实现，不自动生成。当然对于readonly的属性只需要提供getter即可。
如果都没有写@synthesize和@dynamic，那么默认的就是@synthesize ;

### 什么时候不会使用自动合成？

- 同时重写了setter和getter时。
- 重写了只读属性的getter时。
- 使用了@dynamic时。
- 在@protocol中定义的所有属性。
- 在category中定义的所有属性。
- 重载的属性。


## weak、copy、strong、assgin分别用在什么地方？

什么情况下会使用weak关键字？

在ARC中，出现循环引用的时候，会使用weak关键字。
自身已经对它进行了一次强引用，没有必要再强调引用一次。
assgin适用于基本的数据类型，比如NSInteger、BOOL等。

NSString、NSArray、NSDictionary等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；

除了上面的三种情况，剩下的就使用strong来进行修饰。

## 为什么NSString、NSDictionary、NSArray要使用copy修饰符呢？

- 非集合类对象的copy：
```objectivec
[immutableObject copy]; //浅复制
[immutableObject mutableCopy]; //深复制
[mutableObject copy]; //深复制
[mutableObject mutableCopy]; //深复制
```
- 集合类对象的copy：
```objectivec
[immutableObject copy]; //浅复制
[immutableObject mutableCopy]; //单层深复制
[mutableObject copy]; //深复制
[mutableObject mutableCopy]; //深复制
```
是因为它们有对应的可变类型，它们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性时拷贝一份。
