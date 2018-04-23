# 常见面试题

问题1：objc在向一个对象发送消息时，发生了什么？

根据对象的 isa 指针找到类对象 id，在查询类对象里面的 methodLists 方法函数列表，如果没有在找到到，在沿着 superClass ,寻找父类，再在父类 methodLists 方法列表里面查询，最终找到 SEL ,根据 id 和 SEL 确认 IMP（指针函数）,再发送消息。

问题2：什么时候会报unrecognized selector错误？iOS有哪些机制来避免走到这一步？

当发送消息的时候，我们会根据类里面的 methodLists 列表去查询我们要调用的SEL，当查询不到的时候，我们会一直沿着父类查询，当最终查询不到的时候我们会报 unrecognized selector 错误，当系统查询不到方法的时候，会调用 +(BOOL)resolveInstanceMethod:(SEL)sel 动态解析的方法来给我一次机会来添加，调用不到的方法。或者我们可以再次使用 -(id)forwardingTargetForSelector:(SEL)aSelector 重定向的方法来告诉系统，该调用什么方法，一来保证不会崩溃。


问题3：能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

1、不能向编译后得到的类增加实例变量
2、能向运行时创建的类中添加实例变量。
【解释】：1. 编译后的类已经注册在 runtime 中,类结构体中的 objc_ivar_list 实例变量的链表和 instance_size 实例变量的内存大小已经确定，runtime会调用 class_setvarlayout 或 class_setWeaklvarLayout 来处理strong weak 引用.所以不能向存在的类中添加实例变量。2. 运行时创建的类是可以添加实例变量，调用class_addIvar函数. 但是的在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前,原因同上.


问题4：runtime如何实现weak变量的自动置nil？
解答： runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。


问题5：给类添加一个属性后，在类结构体里哪些元素会发生变化？

instance_size ：实例的内存大小；
objc_ivar_list *ivars:属性列表



















