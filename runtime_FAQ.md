# 常见问题

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
3. 第三步：

