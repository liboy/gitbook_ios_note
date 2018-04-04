# 消息发送

>官方文档中 
messages aren’t bound to method implementations until Runtime。
消息直到运行时才会与方法实现进行绑定。

## objc_msgSend函数

当某个对象使用语法[receiver message]来调用某个方法时，其实[receiver message]被编译器转化为：
```c
id objc_msgSend ( id self, SEL op, ... );
```
现在让我们看一下objc_msgSend它具体是如何发送消息：
![](/assets/2.gif)

1. 首先根据receiver对象的isa指针获取它对应的class
- 检测这个 selector 的 target 是不是 nil，Objc 允许我们对一个 nil 对象执行任何方法不会 Crash，因为运行时会被忽略掉。
- 优先在class的cache查找message方法，找不到就找类的方法列表(methodLists)中是否有对应的方法。
- 如果没有在class找到，再到super_class查找，一直找到 NSObject 类为止
- 一旦找到message这个方法，就执行它实现的IMP，如果还找不到，就进入**动态方法解析**了，后面会提到。

在消息的传递中，编译器会根据情况在 `objc_msgSend` ， `objc_msgSend_stret` ， `objc_msgSendSuper` ， `objc_msgSendSuper_stret` 这四个方法中选择一个调用。
- 消息传递给父类，调用名字带有 Super 的函数，
- 消息返回值是数据结构而不是简单值时，会调用名字带有 stret 的函数。

### 隐藏参数self和_cmd

>疑问：我们经常用到关键字 self ，但是 self 是如何获取当前方法的对象呢？

这是 Runtime 系统的作用，self 实在方法运行时被动态传入的。
当 objc_msgSend 找到方法对应实现时，它将直接调用该方法实现，并将消息中所有参数都传递给方法实现，同时，它还将传递两个隐藏参数：

- 接受消息的对象(self 所指向的内容，当前方法的对象指针)
- 方法选择器(_cmd 指向的内容，当前方法的 SEL 指针)

因为在源代码方法的定义中，我们并没有发现这两个参数的声明。它们时在代码被编译时被插入方法实现中的。尽管这些参数没有被明确声明，在源代码中我们仍然可以引用它们。

### self与super
>证明：输出相同，都只能获取当前类的类型名
```
NSLog(@"%@", NSStringFromClass([self class]));
NSLog(@"%@", NSStringFromClass([super class]));
```

当调用[self class]方法时，会转化为objc_msgSend函数，这个函数定义如下：
```
id objc_msgSend(id self, SEL op, ...)
```
这时会从当前Son类的方法列表中查找，如果没有，就到父类查找，还是没有，最后在NSObject类查找到。我们可以从NSObject.mm文件中看到- (Class)class的实现：
```c
- (Class)class {
    return object_getClass(self);
}
```
当调用[super class]方法时，会转化为objc_msgSendSuper，这个函数定义如下：
```c
id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
```
objc_msgSendSuper函数第一个参数super的数据类型是一个指向objc_super的结构体，从message.h文件中查看它的定义：
objc_super结构体含两个成员，
    - receiver = self 表示某个类的实例。
    - super_class = (id)class_getSuperclass(objc_getClass("self.name"))
 表示当前类的父类。

底层编译器将代码转换为:
```c
objc_msgSend(objc_super->receiver, @selector(class))
```
去调用，与[self class]调用相同。

### 获取方法地址methodForSelector

NSObject 类中有一个实例方法：methodForSelector，你可以用它来获取某个方法选择器对应的 IMP ，举个例子：
```c
void (*setter)(id, SEL, BOOL);
int i;

setter = (void (*)(id, SEL, BOOL))[target
    methodForSelector:@selector(setFilled:)];
for ( i = 0 ; i < 1000 ; i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```
当方法被当做函数调用时，两个隐藏参数也必须明确给出，上面的例子调用了1000次函数，你也可以尝试给 target 发送1000次 setFilled: 消息会花多久。

虽然可以更高效的调用方法，但是这种做法很少用，除非时需要持续大量重复调用某个方法的情况，才会选择使用以免消息发送泛滥。

注意：methodForSelector:方法是由 Runtime 系统提供的，而不是 Objc 自身的特性