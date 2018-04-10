# RunLoop应用

## AutoreleasePool

App启动后，苹果在主线程 RunLoop 里注册了两个 Observer管理和维护AutoreleasePool，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()，打印currentRunLoop可以看到。

```c
<CFRunLoopObserver 0x6080001246a0 [0x101f81df0]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
<CFRunLoopObserver 0x608000124420 [0x101f81df0]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
```

- 第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

- 第二个 Observer 监视了两个事件，这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后： 
    - BeforeWaiting(准备进入休眠)时调用`_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()` 释放旧的池并创建新池；
    - Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。
    
![](./images/自动释放池.jpg)    
    
主线程的其他操作通常均在这个AutoreleasePool之内（main函数中），以尽可能减少内存维护操作(当然你如果需要显式释放【例如循环】时可以自己创建AutoreleasePool否则一般不需要自己创建)。