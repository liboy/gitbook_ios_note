# runtime相关术语和数据结构

要想全面了解 Runtime 机制，我们必须先了解 Runtime 的一些术语，他们都对应着数据结构。

## SEL

它是selector在 Objc 中的表示(Swift 中是 Selector 类)。selector 是方法选择器，其实作用就和名字一样，日常生活中，我们通过人名辨别谁是谁，注意 Objc 在相同的类中不会有命名相同的两个方法。selector 对方法名进行包装，以便找到对应的方法实现。它的数据结构是：
``` c
typedef struct objc_selector *SEL;
```
我们可以看出它是个映射到方法的 C 字符串，你可以通过 Objc 编译器器命令@selector() 或者 Runtime 系统的 sel_registerName 函数来获取一个 SEL 类型的方法选择器。
>注意：不同类中相同名字的方法所对应的 selector 是相同的，由于变量的类型不同，所以不会导致它们调用方法实现混乱。

## id

id 是一个参数类型，它是指向某个类的实例的指针。定义如下：
``` c
typedef struct objc_object *id;
struct objc_object { Class isa; };
```
以上定义，看到 objc_object 结构体包含一个 isa 指针，根据 isa 指针就可以找到对象所属的类。

>注意：
isa 指针在代码运行时并不总指向实例对象所属的类型，所以不能依靠它来确定类型，要想确定类型还是需要用对象的 `-class` 方法。

>PS:KVO 的实现机理就是将被观察对象的 isa 指针指向一个中间类而不是真实类型，详见:KVO章节。

## Class
```c
typedef struct objc_class *Class;
```
Class 其实是指向 objc_class 结构体的指针。objc_class 的数据结构如下：
```c
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```
从 objc_class 可以看到，一个运行时类中关联了它的父类指针、类名、成员变量、方法、缓存以及附属的协议。

其中 objc_ivar_list 和 objc_method_list 分别是成员变量列表和方法列表：

// 成员变量列表
struct objc_ivar_list {
    int ivar_count                                           OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

// 方法列表
struct objc_method_list {
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}
由此可见，我们可以动态修改 *methodList 的值来添加成员方法，这也是 Category 实现的原理，同样解释了 Category 不能添加属性的原因。这里可以参考下美团技术团队的文章：深入理解 Objective-C: Category。

objc_ivar_list 结构体用来存储成员变量的列表，而 objc_ivar 则是存储了单个成员变量的信息；同理，objc_method_list 结构体存储着方法数组的列表，而单个方法的信息则由 objc_method 结构体存储。

值得注意的时，objc_class 中也有一个 isa 指针，这说明 Objc 类本身也是一个对象。为了处理类和对象的关系，Runtime 库创建了一种叫做 Meta Class(元类) 的东西，类对象所属的类就叫做元类。Meta Class 表述了类对象本身所具备的元数据。

我们所熟悉的类方法，就源自于 Meta Class。我们可以理解为类方法就是类对象的实例方法。每个类仅有一个类对象，而每个类对象仅有一个与之相关的元类。

当你发出一个类似 [NSObject alloc](类方法) 的消息时，实际上，这个消息被发送给了一个类对象(Class Object)，这个类对象必须是一个元类的实例，而这个元类同时也是一个根元类(Root Meta Class)的实例。所有元类的 isa 指针最终都指向根元类。

所以当 [NSObject alloc] 这条消息发送给类对象的时候，运行时代码 objc_msgSend() 会去它元类中查找能够响应消息的方法实现，如果找到了，就会对这个类对象执行方法调用。


 
上图实现是 super_class 指针，虚线时 isa 指针。而根元类的父类是 NSObject，isa指向了自己。而 NSObject 没有父类。

最后 objc_class 中还有一个 objc_cache ，缓存，它的作用很重要，后面会提到。