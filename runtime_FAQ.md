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


