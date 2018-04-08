# 常见问题

## Method Swizzling（黑魔法）
- 在没有一个类的实现源码时，想改变其中一个方法的实现，除了继承重写和借助类别重命名方法之外，还有更灵活的方法Method Swizzling。
- Method Swizzling指的shi改变一个已存在的选择器对应的实现过程，OC中方法的调用能够在运行时，通过改变类的调度表（dispach table）中选择器到最终函数间的映射关系
- 在OC中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字，利用OC的动态特性，可以实现在运行时偷换selector对应的方法实现。
- 每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系，IMP有点类似函数指针，指向具体的Method实现。
- 可以利用`method_exchangeImplementations`来交换两个方法的IMP；`class_replaceMethod`来修改类；`method_setImplementation`来直接设置某个方法的IMP。
- 归根结底，就是偷换了selector的IMP