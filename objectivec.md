# Objective-C



## @property、@synthesize和@dynamic分别有什么作用？
```objectivec
@property = ivar(实例变量) + getter(取方法) + setter(存方法);
```
- ivar、getter、setter是编译器自动合成的，通过`@synthesize`指定，若不指定，默认为 `@synthesize propertyName = _propertyName`

- `@synthesize`：是如果你没有手动实现getter与setter方法，那么编译器就会自动为你加上这两个方法。

- `@dynamic`：告诉编译器，getter与setter方法由用户自己实现，不自动生成。当然对于readonly的属性只需要提供getter即可。
如果都没有写@synthesize和@dynamic，那么默认的就是@synthesize var = _var;

### 什么时候不会使用自动合成？

- 同时重写了setter和getter时。
- 重写了只读属性的getter时。
- 使用了@dynamic时。
- 在@protocol中定义的所有属性。
- 在category中定义的所有属性。
- 重载的属性。

注意点：
在category中使用@property也是只会生成getter和setter方法的声明，如果真的需要给category增加属性的实现，需要借助于运行时的两个函数：objc_setAssociatedObject和objc_getAssociatedObject。
在protocol中使用property只会生成setter和getter方法声明，使用属性的目的是希望遵守我协议的对象能够实现该属性。
weak、copy、strong、assgin分别用在什么地方？

什么情况下会使用weak关键字？

在ARC中，出现循环引用的时候，会使用weak关键字。
自身已经对它进行了一次强引用，没有必要再强调引用一次。
assgin适用于基本的数据类型，比如NSInteger、BOOL等。

NSString、NSArray、NSDictionary等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；

除了上面的三种情况，剩下的就使用strong来进行修饰。

为什么NSString、NSDictionary、NSArray要使用copy修饰符呢？

要搞清楚这个问题，我们先来弄明白深拷贝与浅拷贝的区别，以非集合类与集合类两种情况来进行说明下，先看非集合类的情况，代码如下：

NSString *name = @"国士梅花";
NSString *newName = [name copy];
NSLog(@"name memory address: %p newName memory address: %p", name, newName);
运行之后，输出的信息如下：

name memory address: 0x10159f758 newName memory address: 0x10159f758
可以看出复制过后，内存地址是一样的，没有发生变化，这就是浅复制，只是把指针地址复制了一份。我们改下代码改成[name mutableCopy]，此时日志的输出信息如下：

name memory address: 0x101b72758 newName memory address: 0x608000263240
我们看到内存地址发生了变化，并且newName的内存地址的偏移量比name的内存地址要大许多，由此可见经过mutableCopy操作之后，复制到堆区了，这就是深复制了，深复制就是内容也就进行了拷贝。

上面的都是不可变对象，在看下可变对象的情况，代码如下：

NSMutableString *name = [[NSMutableString alloc] initWithString:@"国士梅花"];
NSMutableString *newName = [name copy];
NSLog(@"name memory address: %p newName memory address: %p", name, newName);
运行之后日志输出信息如下：

name memory address: 0x600000076e40 newName memory address: 0x6000000295e0
从上面可以看出copy之后，内存地址不一样，且都存储在堆区了，这是深复制，内容也就进行拷贝。在把代码改成[name mutableCopy]，此时日志的输出信息如下：

name memory address: 0x600000077380 newName memory address: 0x6000000776c0
可以看出可变对象copy与mutableCopy的效果是一样的，都是深拷贝。

总结：对于非集合类对象的copy操作如下：

[immutableObject copy]; //浅复制
[immutableObject mutableCopy]; //深复制
[mutableObject copy]; //深复制
[mutableObject mutableCopy]; //深复制
采用同样的方法可以验证集合类对象的copy操作如下：

[immutableObject copy]; //浅复制
[immutableObject mutableCopy]; //单层深复制
[mutableObject copy]; //深复制
[mutableObject mutableCopy]; //深复制
对于NSString、NSDictionary、NSArray等经常使用copy关键字，是因为它们有对应的可变类型：NSMutableString、NSMutableDictionary、NSMutableArray，它们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性时拷贝一份。

作者：国士无双A
链接：https://www.jianshu.com/p/3c0dd2cbb509
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。