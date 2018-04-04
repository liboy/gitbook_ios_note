# 消息发送

>官方文档中 
messages aren’t bound to method implementations until Runtime。
消息直到运行时才会与方法实现进行绑定。

## objc_msgSend函数

当某个对象使用语法[receiver message]来调用某个方法时，其实[receiver message]被编译器转化为：

id objc_msgSend ( id self, SEL op, ... );
现在让我们看一下objc_msgSend它具体是如何发送消息：

首先根据receiver对象的isa指针获取它对应的class；
优先在class的cache查找message方法，如果找不到，再到methodLists查找；
如果没有在class找到，再到super_class查找；
一旦找到message这个方法，就执行它实现的IMP。