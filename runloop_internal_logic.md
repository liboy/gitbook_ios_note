# 内部实现逻辑

## RunLoop 的内部逻辑

根据苹果在文档里的说明，RunLoop 内部的逻辑大致如下:

![](/assets/runloop5.png)

RunLoop的事件队列,官方描述[看这里](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW23)

>1. 通知观察者run loop已经启动
- 通知观察者任何即将要开始的定时器
- 通知观察者任何即将启动的非基于端口的源
- 启动任何准备好的非基于端口的源
- 如果基于端口的源准备好并处于等待状态，立即启动；并进入步骤9。
- 通知观察者线程进入休眠
- 将线程置于休眠直到任一下面的事件发生：
    - 某一事件到达基于端口的源
    - 定时器启动
    - Run loop设置的时间已经超时
    - run loop被显式唤醒
- 通知观察者线程将被唤醒。
- 处理未处理的事件
    - 如果用户定义的定时器启动，处理定时器事件并重启run loop。进入步骤2
    - 如果输入源启动，传递相应的消息
    - 如果run loop被显式唤醒而且时间还没超时，重启run loop。进入步骤2
- 通知观察者run loop结束。

其内部代码整理如下 ：
```c
/// 用DefaultMode启动
void CFRunLoopRun(void) {
    CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
}
  
/// 用指定的Mode启动，允许设置RunLoop超时时间
int CFRunLoopRunInMode(CFStringRef modeName, CFTimeInterval seconds, Boolean stopAfterHandle) {
    return CFRunLoopRunSpecific(CFRunLoopGetCurrent(), modeName, seconds, returnAfterSourceHandled);
}
  
/// RunLoop的实现
int CFRunLoopRunSpecific(runloop, modeName, seconds, stopAfterHandle) {
     
    /// 首先根据modeName找到对应mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(runloop, modeName, false);
    /// 如果mode里没有source/timer/observer, 直接返回。
    if (__CFRunLoopModeIsEmpty(currentMode)) return;
     
    /// 1. 通知 Observers: RunLoop 即将进入 loop。
    __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopEntry);
     
    /// 内部函数，进入loop
    __CFRunLoopRun(runloop, currentMode, seconds, returnAfterSourceHandled) {
         
        Boolean sourceHandledThisLoop = NO;
        int retVal = 0;
        do {
  
            /// 2. 通知 Observers: RunLoop 即将触发 Timer 回调。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeTimers);
            /// 3. 通知 Observers: RunLoop 即将触发 Source0 (非port) 回调。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeSources);
            /// 执行被加入的block
            __CFRunLoopDoBlocks(runloop, currentMode);
             
            /// 4. RunLoop 触发 Source0 (非port) 回调。
            sourceHandledThisLoop = __CFRunLoopDoSources0(runloop, currentMode, stopAfterHandle);
            /// 执行被加入的block
            __CFRunLoopDoBlocks(runloop, currentMode);
  
            /// 5. 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
            if (__Source0DidDispatchPortLastTime) {
                Boolean hasMsg = __CFRunLoopServiceMachPort(dispatchPort, &msg)
                if (hasMsg) goto handle_msg;
            }
             
            /// 通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
            if (!sourceHandledThisLoop) {
                __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
            }
             
            /// 7. 调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
            ///  一个基于 port 的Source 的事件。
            ///  一个 Timer 到时间了
            ///  RunLoop 自身的超时时间到了
            ///  被其他什么调用者手动唤醒
            __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort) {
                mach_msg(msg, MACH_RCV_MSG, port); // thread wait for receive msg
            }
  
            /// 8. 通知 Observers: RunLoop 的线程刚刚被唤醒了。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopAfterWaiting);
             
            /// 收到消息，处理消息。
            handle_msg:
  
            /// 9.1 如果一个 Timer 到时间了，触发这个Timer的回调。
            if (msg_is_timer) {
                __CFRunLoopDoTimers(runloop, currentMode, mach_absolute_time())
            } 
  
            /// 9.2 如果有dispatch到main_queue的block，执行block。
            else if (msg_is_dispatch) {
                __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
            } 
  
            /// 9.3 如果一个 Source1 (基于port) 发出事件了，处理这个事件
            else {
                CFRunLoopSourceRef source1 = __CFRunLoopModeFindSourceForMachPort(runloop, currentMode, livePort);
                sourceHandledThisLoop = __CFRunLoopDoSource1(runloop, currentMode, source1, msg);
                if (sourceHandledThisLoop) {
                    mach_msg(reply, MACH_SEND_MSG, reply);
                }
            }
             
            /// 执行加入到Loop的block
            __CFRunLoopDoBlocks(runloop, currentMode);
             
  
            if (sourceHandledThisLoop && stopAfterHandle) {
                /// 进入loop时参数说处理完事件就返回。
                retVal = kCFRunLoopRunHandledSource;
            } else if (timeout) {
                /// 超出传入参数标记的超时时间了
                retVal = kCFRunLoopRunTimedOut;
            } else if (__CFRunLoopIsStopped(runloop)) {
                /// 被外部调用者强制停止了
                retVal = kCFRunLoopRunStopped;
            } else if (__CFRunLoopModeIsEmpty(runloop, currentMode)) {
                /// source/timer/observer一个都没有了
                retVal = kCFRunLoopRunFinished;
            }
             
            /// 如果没超时，mode里没空，loop也没被停止，那继续loop。
        } while (retVal == 0);
    }
     
    /// 10. 通知 Observers: RunLoop 即将退出。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
}
```
可以看到，实际上 RunLoop 就是这样一个函数，其内部是一个 do-while 循环。当你调用 CFRunLoopRun() 时，线程就会一直停留在这个循环里；直到超时或被手动停止，该函数才会返回。

这张图更详细描述了上面Runloop的核心流程：
![](/assets/runloop6.png)

>注意：
- 是_黄色_区域的消息处理中并不包含source0，因为它在循环开始之初就会处理
- CFRunLoopPerformBlock尽管在上图中作为唤醒机制有所体现，但事实上执行只是入队，等待下次RunLoop运行才会执行，而如果需要立即执行则必须调用CFRunLoopWakeUp。

## RunLoop 的底层实现
从上面代码的第7步可以看到，RunLoop 的核心是基于 mach port 的，其进入休眠时调用的函数是 `mach_msg()`。

```
/// 7. 调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
///  一个基于 port 的Source 的事件。
///  一个 Timer 到时间了
///  RunLoop 自身的超时时间到了
///  被其他什么调用者手动唤醒
__CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort) {
    mach_msg(msg, MACH_RCV_MSG, port); // thread wait for receive msg
}
```

![OSX/iOS 这个核心的架构](/assets/runloop7.png)

- Mach 是 Darwin 的核心，可以说是内核的核心，提供了进程间通信（IPC）、处理器调度等基础服务。在 Mach 中，进程、线程间的通信是以消息的方式来完成的，消息在两个 Port 之间进行传递（这也正是 Source1 之所以称之为 Port-based Source 的原因，因为它就是依靠系统发送消息到指定的Port来触发的）。消息的发送和接收使用<mach/message.h>中的mach_msg()函数

- 为了实现消息的发送和接收，`mach_msg()` 函数实际上是调用了一个 Mach 陷阱 (trap)，即函数`mach_msg_trap()`，陷阱这个概念在 Mach 中等同于系统调用。当你在用户态调用 `mach_msg_trap()` 时会触发陷阱机制，切换到内核态；内核态中内核实现的 mach_msg() 函数会完成实际的工作，如下图：
![](/assets/runloop8.png)
例如你在模拟器里跑起一个 iOS 的 App，然后在 App 静止时点击暂停，你会看到主线程调用栈是停留在 `mach_msg_trap()` 这个地方


关于具体 mach port 发送信息可以查看：NSHipster这篇[文章](http://nshipster.com/inter-process-communication/)，或[中文翻译](http://segmentfault.com/a/1190000002400329) 。