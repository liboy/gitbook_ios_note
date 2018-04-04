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

1. 首先根据receiver对象的isa指针获取它对应的class；
2. 优先在class的cache查找message方法，如果找不到，再到methodLists查找；
3. 如果没有在class找到，再到super_class查找；
4. 一旦找到message这个方法，就执行它实现的IMP。

首先检测这个 selector 是不是要忽略。比如 Mac OS X 开发，有了垃圾回收就不理会 retain，release 这些函数。
- 检测这个 selector 的 target 是不是 nil，Objc 允许我们对一个 nil 对象执行任何方法不会 Crash，因为运行时会被忽略掉。
- 如果上面两步都通过了，那么就开始查找这个类的实现 IMP，先从 cache 里查找，如果找到了就运行对应的函数去执行相应的代码。
- 如果 cache 找不到就找类的方法列表中是否有对应的方法。
- 如果类的方法列表中找不到就到父类的方法列表中查找，一直找到 NSObject 类为止。
- 如果还找不到，就要开始进入**动态方法解析**了，后面会提到。