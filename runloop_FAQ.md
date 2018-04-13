# 常见面试题

<!--sec data-title="问题1：runloop是来做什么的？runloop和线程有什么关系？主线程默认开启了runloop么？子线程呢？
" data-id="section0" data-show=true ces-->

runloop: 从字面意思看：运行循环、跑圈，其实它内部就是do-while循环，在这个循环内部不断地处理各种任务（比如Source、Timer、Observer）事件。runloop和线程的关系：一个线程对应一个RunLoop，主线程的RunLoop默认创建并启动，子线程的RunLoop需手动创建且手动启动（调用run方法）。RunLoop只能选择一个Mode启动，如果当前Mode中没有任何Source(Sources0、Sources1)、Timer，那么就直接退出RunLoop。  

<!--endsec--> 

<!--sec data-title="问题2：runloop的mode是用来做什么的？有几种mode？
" data-id="section2" data-show=true ces-->
 
解答： model:是runloop里面的运行模式，不同的模式下的runloop处理的事件和消息有一定的差别。系统默认注册了5个Mode:（1）kCFRunLoopDefaultMode: App的默认 Mode，通常主线程是在这个 Mode 下运行的。（2）UITrackingRunLoopMode: 界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响。（3）UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用。（4）GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到。（5）kCFRunLoopCommonModes: 这是一个占位的 Mode，没有实际作用。注意iOS 对以上5中model进行了封装 NSDefaultRunLoopMode、NSRunLoopCommonModes
<!--endsec--> 


<!--sec data-title="问题3：为什么把NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环以后，滑动scrollview的时候NSTimer却不动了？
" data-id="section3" data-show=true ces-->

NSTimer对象是在 NSDefaultRunLoopMode下面调用消息的，但是当我们滑动scrollview的时候，NSDefaultRunLoopMode模式就自动切换到UITrackingRunLoopMode模式下面，却不可以继续响应nstime发送的消息。所以如果想在滑动scrollview的情况下面还调用nstime的消息，我们可以把nsrunloop的模式更改为NSRunLoopCommonModes.

<!--endsec--> 

<!--sec data-title="问题4：苹果是如何实现Autorelease Pool的？
" data-id="section4" data-show=true ces-->
 
解答： Autorelease Pool作用：缓存池，可以避免我们经常写relase的一种方式。其实就是延迟release，将创建的对象，添加到最近的autoreleasePool中，等到autoreleasePool作用域结束的时候，会将里面所有的对象的引用计数器 - autorelease.
<!--endsec--> 

