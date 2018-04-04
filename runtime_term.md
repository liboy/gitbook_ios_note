# runtime相关术语和数据结构

要想全面了解 Runtime 机制，我们必须先了解 Runtime 的一些术语，他们都对应着数据结构。

## SEL

SEL是函数objc_msgSend第二个参数的数据类型，表示方法选择器，在objc.h文件数据结构是：
``` c
typedef struct objc_selector *SEL;
```
- 它是个映射到方法的 C 字符串，你可以通过 Objc 编译器器命令@selector() 或者 Runtime 系统的 sel_registerName 函数来获取一个 SEL 类型的方法选择器。
- 可以通过NSString* NSStringFromSelector(SEL aSelector)方法将SEL转化为字符串
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

>PS:KVO 的实现机理就是将被观察对象的 isa 指针指向一个中间类而不是真实类型。

## Class
Class 其实是指向 objc_class 结构体的指针。
```c
typedef struct objc_class *Class;
```
打开`runtime.h`文件objc_class 的数据结构如下：
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
>注意：OBJC2_UNAVAILABLE是一个Apple对Objc系统运行版本进行约束的宏定义，主要为了兼容非Objective-C 2.0的遗留版本，但我们仍能从中获取一些有用信息。

从 objc_class 可以看到，一个运行时类中关联了它的父类指针、类名、成员变量、方法、缓存以及附属的协议。

- **isa**表示一个Class对象的Class，也就是Meta Class。在面向对象设计中，一切都是对象，Class在设计中本身也是一个对象。我们会在objc-runtime-new.h文件找到证据，发现objc_class有以下定义：
```c
struct objc_class : objc_object {
  // Class ISA;
  Class superclass;
  cache_t cache;             // formerly cache pointer and vtable
  class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

  ......
}
```
由此可见，结构体objc_class也是继承objc_object，说明Class在设计中本身也是一个对象。
#### Meta Class(元类)
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
```c
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
```c
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
