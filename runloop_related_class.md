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


>注意：Source/Timer/Observer 被统称为 mode item，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。



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

- Source0 (负责App内部事件，由App负责管理触发，例如UITouch事件) 只包含了一个回调（函数指针），它不能主动触发事件。使用时，你需要先调用`CFRunLoopSourceSignal(source)`，将这个 Source 标记为待处理，然后手动调用 `CFRunLoopWakeUp(runloop)` 来唤醒 RunLoop，就会处理并调用事件处理方法。
- Source1 包含了一个 `mach_port` 和一个回调（函数指针），可以监听系统端口和其他线程相互发送消息，能主动唤醒 RunLoop(由操作系统内核进行管理，例如CFMessagePort消息)。

### 输入事件来源

Run loop接收输入事件来自两种不同的来源：
- 1.输入源（input source）
- 2.定时源（timer source）

两种源都使用程序的某一特定的处理例程来处理到达的事件。


当你创建输入源，你需要将其分配给run loop中的一个或多个模式。模式只会在特定事件影响监听的源。大多数情况下，run loop运行在默认模式下，但是你也可以使其运行在自定义模式。若某一源在当前模式下不被监听，那么任何其生成的消息只在run loop运行在其关联的模式下才会被传递。

![Runloop的结构和输入源类型](./assets/runloop.jpg) 

#### 1.输入源（input source）

传递异步事件，通常消息来自于其他线程或程序。输入源传递异步消息给相应的处理例程，并调用runUntilDate:方法来退出(在线程里面相关的NSRunLoop对象调用)。

##### 1.1基于端口的输入源

基于端口的输入源由内核自动发送。

Cocoa和Core Foundation内置支持使用端口相关的对象和函数来创建的基于端口的源。例如，在Cocoa里面你从来不需要直接创建输入源。你只要简单的创建端口对象，并使用NSPort的方法把该端口添加到run loop。端口对象会自己处理创建和配置输入源。

在Core Foundation，你必须人工创建端口和它的run loop源。我们可以使用端口相关的函数（CFMachPortRef，CFMessagePortRef，CFSocketRef）来创建合适的对象。下面的例子展示了如何创建一个基于端口的输入源，将其添加到run loop并启动：
```objectc
void createPortSource() {
    
    CFMessagePortRef port = CFMessagePortCreateLocal(kCFAllocatorDefault, CFSTR("com.someport"),myCallbackFunc, NULL, NULL);
    
    CFRunLoopSourceRef source = CFMessagePortCreateRunLoopSource(kCFAllocatorDefault, port, 0);
    
    CFRunLoopAddSource(CFRunLoopGetCurrent(), source, kCFRunLoopCommonModes);
    
    while (pageStillLoading) {
        
        NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
        
        CFRunLoopRun();
        
        [pool release];
        
    }
    
    CFRunLoopRemoveSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);
    
    CFRelease(source);
    
}
```
##### 1.2自定义输入源

自定义的输入源需要人工从其他线程发送。

为了创建自定义输入源，必须使用Core Foundation里面的CFRunLoopSourceRef类型相关的函数来创建。你可以使用回调函数来配置自定义输入源。Core Fundation会在配置源的不同地方调用回调函数，处理输入事件，在源从run loop移除的时候清理它。

除了定义在事件到达时自定义输入源的行为，你也必须定义消息传递机制。源的这部分运行在单独的线程里面，并负责在数据等待处理的时候传递数据给源并通知它处理数据。消息传递机制的定义取决于你，但最好不要过于复杂。创建并启动自定义输入源的示例如下：
```objectc
void createCustomSource()
{
    CFRunLoopSourceContext context = {0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL};

    CFRunLoopSourceRef source = CFRunLoopSourceCreate(kCFAllocatorDefault, 0, &context);

    CFRunLoopAddSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);

    while (pageStillLoading) {

        NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];

        CFRunLoopRun();

        [pool release];

    }

    CFRunLoopRemoveSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);

    CFRelease(source);

}
```
##### 1.3Cocoa上的Selector源

除了基于端口的源，Cocoa定义了自定义输入源，允许你在任何线程执行selector方法。和基于端口的源一样，执行selector请求会在目标线程上序列化，减缓许多在线程上允许多个方法容易引起的同步问题。不像基于端口的源，一个selector执行完后会自动从run loop里面移除。

当在其他线程上面执行selector时，目标线程须有一个活动的run loop。对于你创建的线程，这意味着线程在你显式的启动run loop之前是不会执行selector方法的，而是一直处于休眠状态。

NSObject类提供了类似如下的selector方法：

- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)argwaitUntilDone:(BOOL)wait modes:(NSArray *)array;

 

### 2定时源（timer source）

定时源在预设的时间点同步方式传递消息，这些消息都会发生在特定时间或者重复的时间间隔。定时源则直接传递消息给处理例程，不会立即退出run loop。

需要注意的是，尽管定时器可以产生基于时间的通知，但它并不是实时机制。和输入源一样，定时器也和你的run loop的特定模式相关。如果定时器所在的模式当前未被run loop监视，那么定时器将不会开始直到run loop运行在相应的模式下。类似的，如果定时器在run loop处理某一事件期间开始，定时器会一直等待直到下次run loop开始相应的处理程序。如果run loop不再运行，那定时器也将永远不启动。

创建定时器源有两种方法，

方法一：

NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:4.0

                                                     target:self

                                                   selector:@selector(backgroundThreadFire:) userInfo:nil

                                                    repeats:YES];

    [[NSRunLoop currentRunLoop] addTimer:timerforMode:NSDefaultRunLoopMode];

 

方法二：

[NSTimer scheduledTimerWithTimeInterval:10

                                        target:self

                                       selector:@selector(backgroundThreadFire:)

                                       userInfo:nil

                                       repeats:YES];

- 【Port-Based Sources】：基于端口的源 (对应的是source1)：与内核端口相关，只需要简单的创建端口对象，并使用 NSPort 的方法将端口对象加入到runloop，端口对象会处理创建以及配置输入源对应，Source1和Timer都属于端口事件源，不同的是所有的Timer都共用一个端口`Mode Timer Port`，而每个Source1都有不同的对应端口
- 【Custom Input Sources】：自定义源：使用CFRunLoopSourceRef 类型相关的函数 (线程) 来创建自定义输入源。
- 【Perform Selector Sources】：`performSelector:OnThread:delay:`

## Timer
CFRunLoopTimerRef 是基于时间的触发器，上层对应NSTimer
，它和 NSTimer 是`toll-free bridged` 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

## Observer
```c
struct __CFRunLoopObserver {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFIndex _rlCount;
    CFOptionFlags _activities;      /* immutable */
    CFIndex _order;         /* immutable */
    CFRunLoopObserverCallBack _callout; /* immutable */
    CFRunLoopObserverContext _context;  /* immutable, except invalidation */
};
```
CFRunLoopObserverRef相当于消息循环中的一个监听器，随时通知外部当前RunLoop的运行状态（它包含一个函数指针`_callout`将当前状态及时告诉观察者）。具体的Observer状态如下

```objectc
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
     //创建监听者
     /*
     第一个参数 CFAllocatorRef allocator：分配存储空间 CFAllocatorGetDefault()默认分配
     第二个参数 CFOptionFlags activities：要监听的状态 kCFRunLoopAllActivities 监听所有状态
     第三个参数 Boolean repeats：YES:持续监听 NO:不持续
     第四个参数 CFIndex order：优先级，一般填0即可
     第五个参数 ：回调 两个参数observer:监听者 activity:监听的事件
     */
     /*
     所有事件
     typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity{
        kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
        kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
        kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
        kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
        kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
        kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
    };
     */
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
        switch (activity) {
            case kCFRunLoopEntry:
                NSLog(@"RunLoop进入");
                break;
            case kCFRunLoopBeforeTimers:
                NSLog(@"RunLoop要处理Timers了");
                break;
            case kCFRunLoopBeforeSources:
                NSLog(@"RunLoop要处理Sources了");
                break;
            case kCFRunLoopBeforeWaiting:
                NSLog(@"RunLoop要休息了");
                break;
            case kCFRunLoopAfterWaiting:
                NSLog(@"RunLoop醒来了");
                break;
            case kCFRunLoopExit:
                NSLog(@"RunLoop退出了");
                break;
                
            default:
                break;
        }
    });
    
    // 给RunLoop添加监听者
    /*
     第一个参数 CFRunLoopRef rl：要监听哪个RunLoop,这里监听的是主线程的RunLoop
     第二个参数 CFRunLoopObserverRef observer 监听者
     第三个参数 CFStringRef mode 要监听RunLoop在哪种运行模式下的状态
     */
    CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);
     /*
     CF的内存管理（Core Foundation）
     凡是带有Create、Copy、Retain等字眼的函数，创建出来的对象，都需要在最后做一次release
     GCD本来在iOS6.0之前也是需要我们释放的，6.0之后GCD已经纳入到了ARC中，所以我们不需要管了
     */
    CFRelease(observer);
}
```
### Call out
在开发过程中几乎所有的操作都是通过Call out进行回调的(无论是Observer的状态通知还是Timer、Source的处理)，而系统在回调时通常使用如下几个函数进行回调(换句话说你的代码其实最终都是通过下面几个函数来负责调用的，即使你自己监听Observer也会先调用下面的函数然后间接通知你，所以在调用堆栈中经常看到这些函数)：

```c
static void __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__();
static void __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__();
```
例如在控制器的touchBegin中打入断点查看堆栈（由于UIEvent是Source0，所以可以看到一个Source0的Call out函数`CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION`调用）：
![](/assets/runloop4.png)




