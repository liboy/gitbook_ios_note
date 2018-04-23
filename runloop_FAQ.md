# 常见面试题

问题1：runloop是来做什么的？runloop和线程有什么关系？主线程默认开启了runloop么？子线程呢？

runloop: 从字面意思看：运行循环、跑圈，其实它内部就是do-while循环，在这个循环内部不断地处理各种任务（比如Source、Timer、Observer）事件。runloop和线程的关系：一个线程对应一个RunLoop，主线程的RunLoop默认创建并启动，子线程的RunLoop需手动创建且手动启动（调用run方法）。RunLoop只能选择一个Mode启动，如果当前Mode中没有任何Source(Sources0、Sources1)、Timer，那么就直接退出RunLoop。

问题4：苹果是如何实现Autorelease Pool的？

解答： Autorelease Pool作用：缓存池，可以避免我们经常写relase的一种方式。其实就是延迟release，将创建的对象，添加到最近的autoreleasePool中，等到autoreleasePool作用域结束的时候，会将里面所有的对象的引用计数器 - autorelease.

