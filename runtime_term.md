# runtime相关术语和数据结构

要想全面了解 Runtime 机制，我们必须先了解 Runtime 的一些术语，他们都对应着数据结构。

## SEL

SEL是函数objc_msgSend第二个参数的数据类型，表示方法选择器，在objc.h文件数据结构是：
```objectivec
typedef struct objc_selector *SEL;
```
- 它是个映射到方法的 C 字符串，你可以通过 Objc 编译器器命令`@selector()` 或者 Runtime 系统的 `sel_registerName` 函数来获取一个 `SEL` 类型的方法选择器。
- 可以通过`NSString* NSStringFromSelector(SEL aSelector)`方法将SEL转化为字符串

>注意：不同类中相同名字的方法所对应的 selector 是相同的，由于变量的类型不同，所以不会导致它们调用方法实现混乱。

## id

id 是一个参数类型，它是指向某个类的实例的指针。定义如下：
```objectivec
typedef struct objc_object *id;
struct objc_object { Class isa; };
```
以上定义，看到 `objc_object` 结构体包含一个 `isa` 指针，根据 isa 指针就可以找到对象所属的类。

>注意：
isa 指针在代码运行时并不总指向实例对象所属的类型，所以不能依靠它来确定类型，要想确定类型还是需要用对象的 `- class` 方法。

>PS:KVO 的实现机理就是将被观察对象的 `isa` 指针指向一个中间类而不是真实类型。

## Class

Class 其实是指向 objc_class 结构体的指针。
```objectivec
typedef struct objc_class *Class;
```
打开`runtime.h`文件objc_class 的数据结构如下：
```objectivec
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
>注意：OBJC2_UNAVAILABLE是一个Apple对Objc系统运行版本进行约束的宏定义，主要为了兼容非Objective-C 2.0的遗留版本，但我们仍能从中获取一些有用信息。

从 objc_class 可以看到，一个运行时类中关联了它的父类指针、类名、成员变量、方法、缓存以及附属的协议。

- **isa**表示一个Class对象的Class，也就是Meta Class。在面向对象设计中，一切都是对象，Class在设计中本身也是一个对象。我们会在objc-runtime-new.h文件找到证据，发现objc_class有以下定义：
```objectivec
struct objc_class : objc_object {
  // Class ISA;
  Class superclass;
  cache_t cache;             // formerly cache pointer and vtable
  class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

  ......
}
```
由此可见，结构体objc_class也是继承objc_object，说明Class在设计中本身也是一个对象。
为了处理类和对象的关系，Runtime 库创建了一种叫做 Meta Class(元类) 的东西，类对象所属的类就叫做元类。
 - Meta Class 表述了类对象本身所具备的元数据。
 - 我们所熟悉类方法，就源自于 Meta Class。
 - 类方法就是类对象的实例方法，每个类仅有一个类对象，而每个类对象仅有一个与之相关的元类。
Meta Class也是一个Class，那么它也跟其他Class一样有自己的isa和super_class指针，关系如下：
![](/assets/1.png)
 1. Root class (class)其实就是NSObject，NSObject是没有超类的，所以Root class(class)的superclass指向nil。
 2. 每个Class都有一个isa指针指向唯一的Meta class
 3. Root class(meta)的superclass指向Root class(class)，也就是NSObject，形成一个回路。
 4. 每个Meta class的isa指针都指向Root class (meta)。

- **super_class**表示实例对象对应的父类；

- **name**表示类名

- **ivars**表示多个成员变量，它指向objc_ivar_list结构体。在runtime.h可以看到它的定义：
```objectivec
struct objc_ivar_list {
  int ivar_count                                           OBJC2_UNAVAILABLE;
#ifdef __LP64__
  int space                                                OBJC2_UNAVAILABLE;
#endif
  /* variable length structure */
  struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}
```
objc_ivar_list其实就是一个链表，存储多个objc_ivar，而objc_ivar结构体存储类的单个成员变量信息。

- **methodLists**表示方法列表，它指向objc_method_list结构体的二级指针，可以动态修改*methodLists的值来添加成员方法，也是Category实现原理，同样也解释Category不能添加属性的原因。在runtime.h可以看到它的定义：
```objectivec
struct objc_method_list {
  struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

  int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
  int space                                                OBJC2_UNAVAILABLE;
#endif
  /* variable length structure */
  struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}
```
同理，objc_method_list也是一个链表，存储多个objc_method，而objc_method结构体存储类的某个方法的信息。

- **cache**用来缓存经常访问的方法，它指向objc_cache结构体，后面会重点讲到。
- **protocols**表示类遵循哪些协议。

## Method

Method表示类中的某个方法，在runtime.h文件中找到它的定义：

```objectivec
/// An opaque type that represents a method in a class definition.
typedef struct objc_method *Method;
struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}
```
其实Method就是一个指向objc_method结构体指针，它存储了
- 方法名(method_name)
- 方法类型(method_types)
- 方法实现(method_imp)，method_imp的数据类型是IMP，它是一个函数指针

## Ivar

Ivar表示类中的实例变量，在runtime.h文件中找到它的定义：
```objectivec
/// An opaque type that represents an instance variable.
typedef struct objc_ivar *Ivar;

struct objc_ivar {
    char *ivar_name                                          OBJC2_UNAVAILABLE;
    char *ivar_type                                          OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}
```
Ivar其实就是一个指向objc_ivar结构体指针，它包含了变量名(ivar_name)、变量类型(ivar_type)等信息。

## IMP

在上面讲Method时就说过，IMP本质上就是一个函数指针，指向方法的实现，在objc.h找到它的定义：
```c
/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id (*IMP)(id, SEL, ...); 
#endif
```
当你向某个对象发送一条信息，可以由这个函数指针来指定方法的实现，它最终就会执行那段代码，这样可以绕开消息传递阶段而去执行另一个方法实现。

## Cache

在runtime.h文件看看它的定义：
```objectivec
typedef struct objc_cache *Cache                             OBJC2_UNAVAILABLE;

struct objc_cache {
    unsigned int mask /* total = mask + 1 */                 OBJC2_UNAVAILABLE;
    unsigned int occupied                                    OBJC2_UNAVAILABLE;
    Method buckets[1]                                        OBJC2_UNAVAILABLE;
};
```
Cache其实就是一个存储Method的链表，主要是为了优化方法调用的性能。当对象receiver调用方法message时，首先根据对象receiver的isa指针查找到它对应的类，然后在类的methodLists中搜索方法，如果没有找到，就使用super_class指针到父类中的methodLists查找，一旦找到就调用方法。如果没有找到，有可能消息转发，也可能忽略它。但这样查找方式效率太低，因为往往一个类大概只有20%的方法经常被调用，占总调用次数的80%。所以使用Cache来缓存经常调用的方法，当调用方法时，优先在Cache查找，如果没有找到，再到methodLists查找。

## Property
```objectivec

typedef struct objc_property *Property;
typedef struct objc_property *objc_property_t;//这个更常用
```
可以通过class_copyPropertyList 和 protocol_copyPropertyList 方法获取类和协议中的属性：
```objectivec
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```
>注意：返回的是属性列表，列表中每个元素都是一个 objc_property_t 指针

```objectivec
#import <Foundation/Foundation.h>

@interface Person : NSObject

/** 姓名 */
@property (strong, nonatomic) NSString *name;

/** age */
@property (assign, nonatomic) int age;

/** weight */
@property (assign, nonatomic) double weight;

@end
```
以上是一个 Person 类，有3个属性。让我们用上述方法获取类的运行时属性。
```objectivec
unsigned int outCount = 0;

objc_property_t *properties = class_copyPropertyList([Person class], &outCount);

NSLog(@"%d", outCount);

for (NSInteger i = 0; i < outCount; i++) {
    NSString *name = @(property_getName(properties[i]));
    NSString *attributes = @(property_getAttributes(properties[i]));
    NSLog(@"%@--------%@", name, attributes);
}
```
打印结果如下：
```
test[2321:451525] 3
test[2321:451525] name--------T@"NSString",&,N,V_name
test[2321:451525] age--------Ti,N,V_age
test[2321:451525] weight--------Td,N,V_weight
```
- property_getName 用来查找属性的名称，返回 c 字符串。
- property_getAttributes函数可以获得属性的名字和@encode编码。

>关于类型编码参考：[类型编码](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1) 、[属性的类型编码](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW6)

class_getProperty 和 protocol_getProperty 通过给出属性名在类和协议中获得属性的引用。
```objectivec
objc_property_t class_getProperty(Class cls, const char *name)
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
```
## Method Swizzling（黑魔法）

- 在没有一个类的实现源码时，想改变其中一个方法的实现，除了继承重写和借助类别重命名方法之外，还有更灵活的方法Method Swizzling。
- Method Swizzling指的shi改变一个已存在的选择器对应的实现过程，OC中方法的调用能够在运行时，通过改变类的调度表（dispach table）中选择器到最终函数间的映射关系
- 在OC中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字，利用OC的动态特性，可以实现在运行时偷换selector对应的方法实现。
- 每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系，IMP有点类似函数指针，指向具体的Method实现。
- 可以利用`method_exchangeImplementations`来交换两个方法的IMP；`class_replaceMethod`来修改类；`method_setImplementation`来直接设置某个方法的IMP。
- 归根结底，就是偷换了selector的IMP

如何利用好这个技巧写出简洁、有效且维护性更好的代码。参考：[Method Swizzling 和 AOP 实践](http://tech.glowing.com/cn/method-swizzling-aop/)

## Aspect-Oriented Programming(AOP)

- 类似记录日志、身份验证、缓存等事务非常琐碎，与业务逻辑无关，很多地方都有，又很难抽象出一个模块，这种程序设计问题，业界给它们起了一个名字叫横向关注点(Cross-cutting concern)，
- AOP作用就是分离横向关注点(Cross-cutting concern)来提高模块复用性，它可以在既有的代码添加一些额外的行为(记录日志、身份验证、缓存)而无需修改代码。

## 动态绑定
- 运行时的消息分发机制为动态绑定提供支持
- 当向一个动态类型确定了的对象发送消息是，运行环境系统会通过接受的isa指针定位对象的类，并以此为起点确定被调用的方法，方法和消息是动态绑定的。
- 不必在OC代码做任何工作，就可自动获取动态绑定的好处

## objc_msgForward
是IMP类型，用于消息转发，当向一个对象发送消息，但它并没有实现，_bojc_msgForward会尝试做消息转发。
如果手动调用，将跳过查找IMP的过程，直接触发消息转发，进入如下流程：
1. 第一步：`+ (BOOL)resolveInstanceMethod:(SEL)sel`方法，指定是否动态添加方法，返回NO，则进入下一步；返回YES，则通过class_addMethod函数动态添加方法，消息得到处理，流程完毕。
2. 第二步：`- (id)forwardingTargetForSelector:(SEL)aSelector`方法，这是运行时给我们的第二次机会，用于指定哪个对象响应这个selector,不能指定为self,若返回nil，表示没有响应者，则会进入第三步；若返回某个对象，则会调用该对象的方法。
3. 第三步：`- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`方法，指定方法签名，返回nil，表示不出理；返回签名，进入第四步。
4. 第四步：`- (void)forwardInvocation:(NSInvocation *)anInvocation  `方法，可以通过anInvocation对象做很多处理，比如：修改实现方法、修改响应对象等；若没有实现，进入第五步。
5. 第五步：`- (void)doesNotRecognizeSelector:(SEL)aSelector`作为找不到函数实现的最后一步，NSObject实现这个函数只有一个功能，就是抛出异常。虽然理论上可以重载这个函数实现保证不抛出异常（不调用super实现），但是苹果文档着重提出“一定不能让这个函数就这么结束掉，必须抛出异常”。

具体可查看：[这篇文章](https://www.cnblogs.com/biosli/p/NSObject_inherit_2.html)



