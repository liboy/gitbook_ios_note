# 相关类

Core Foundation 中关于 RunLoop 的5个类

1. CFRunloopRef【RunLoop本身】
2. CFRunloopModeRef【Runloop的运行模式】
3. CFRunloopSourceRef【Runloop要处理的事件源】
4. CFRunloopTimerRef【Timer事件】
5. CFRunloopObserverRef【Runloop的观察者（监听者）】

![](/assets/runloop3.png)
由图中可以得出以下几点：
1. CFRunLoopModeRef代表的是RunLoop的运行模式。
2. 一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。
3. 每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。
4. 如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

>注意：一种Mode中可以有多个Source(事件源，输入源，基于端口事件源例键盘触摸等) Observer(观察者，观察当前RunLoop运行状态) 和Timer(定时器事件源)。但是必须至少有一个Source或者Timer，因为如果Mode为空，RunLoop运行到空模式不会进行空转，就会立刻退出。

## Mode

RunLoop 有五种运行模式，其中常见的有1.2两种

1. kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行
2. UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
3. UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
4. GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
5. kCFRunLoopCommonModes: 这是一个占位用的Mode，作为标记kCFRunLoopDefaultMode和UITrackingRunLoopMode用，并不是一种真正的Mode

## CFRunLoopSourceRef事件源
- Source0：非基于Port的 用于用户主动触发的事件（点击button 或点击屏幕）
- Source1：基于Port的 通过内核和其他线程相互发送消息（与内核相关）
>注意：Source1在处理的时候会分发一些操作给Source0去处理

## CFRunLoopObserverRef

CFRunLoopObserver是观察者，可以监听runLoop的状态改变
监听的状态如下：

typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) { 
kCFRunLoopEntry = (1UL << 0), //即将进入Runloop
kCFRunLoopBeforeTimers = (1UL << 1), //即将处理NSTimer 
kCFRunLoopBeforeSources = (1UL << 2), //即将处理Sources 
kCFRunLoopBeforeWaiting = (1UL << 5), //即将进入休眠 
kCFRunLoopAfterWaiting = (1UL << 6), //刚从休眠中唤醒 
kCFRunLoopExit = (1UL << 7), //即将退出runloop 
kCFRunLoopAllActivities = 0x0FFFFFFFU //所有状态改变};




