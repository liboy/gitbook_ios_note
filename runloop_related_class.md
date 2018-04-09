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

CFRunLoopMode 和 CFRunLoop 的结构大致如下：
```c
struct __CFRunLoopMode {
    CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
  
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```
CFRunLoop对外暴露的管理 Mode 接口只有下面2个:
```c
CFRunLoopAddCommonMode(CFRunLoopRef runloop, CFStringRef modeName);
CFRunLoopRunInMode(CFStringRef modeName, ...);
```
Mode 暴露的管理 mode item 的接口有下面几个：
```c
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```


- _commonModes：一个 mode 可以标记为 `Common` 属性,主线程的 RunLoop 里有两个预置的 Mode：`kCFRunLoopDefaultMode` 和 `UITrackingRunLoopMode`都已经被标记为`Common`属性，当然你也可以通过调用 `CFRunLoopAddCommonMode()` 方法将自定义mode 放到 `kCFRunLoopCommonModes` 组合。

- commonModeItems：存放的source, observer, timer等，在每次 runLoop 运行的时候都会被同步到具有 `Common` 标记的 Modes 里。如：`[[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes]` 就是把timer放到commonModeItems 里。

- 更多系统或框架 Mode查看[这里](http://iphonedevwiki.net/index.php/CFRunLoop)



## Source

CFRunLoopSourceRef 是事件产生的地方。Source有两个版本：Source0 和 Source1。

- Source0 只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用`CFRunLoopSourceSignal(source)`，将这个 Source 标记为待处理，然后手动调用 `CFRunLoopWakeUp(runloop)` 来唤醒 RunLoop，让其处理这个事件。
- Source1 包含了一个 `mach_port` 和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程，其原理在下面会讲到。

![官方Runloop结构图](./assets/runloop.jpg)

图中Input Source Port和前面流程图中的Source0并不对应，而是对应Source1。Source1和Timer都属于端口事件源，不同的是所有的Timer都共用一个端口“Mode Timer Port”，而每个Source1都有不同的对应端口）：


再结合前面RunLoop核心运行流程可以看出Source0(负责App内部事件，由App负责管理触发，例如UITouch事件)和Timer（又叫Timer Source，基于时间的触发器，上层对应NSTimer）是两个不同的Runloop事件源（当然Source0是Input Source中的一类，Input Source还包括Custom Input Source，由其他线程手动发出），RunLoop被这些事件唤醒之后就会处理并调用事件处理方法（CFRunLoopTimerRef的回调指针和CFRunLoopSourceRef均包含对应的回调指针）。
但是对于CFRunLoopSourceRef除了Source0之外还有另一个版本就是Source1，Source1除了包含回调指针外包含一个mach port，和Source0需要手动触发不同，Source1可以监听系统端口和其他线程相互发送消息，它能够主动唤醒RunLoop(由操作系统内核进行管理，例如CFMessagePort消息)。官方也指出可以自定义Source，因此对于CFRunLoopSourceRef来说它更像一种协议，框架已经默认定义了两种实现，如果有必要开发人员也可以自定义，详细情况可以查看官方文档。

## CFRunLoopTimerRef
CFRunLoopTimerRef 是基于时间的触发器，它和 NSTimer 是`toll-free bridged` 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

## CFRunLoopObserverRef
CFRunLoopObserverRef 是观察者，每个 Observer 都包含了一个回调（函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。可以观测的时间点有以下几个：

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```
上面的 Source/Timer/Observer 被统称为 mode item，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。



