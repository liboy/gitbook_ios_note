# 输入事件来源

Run loop接收输入事件来自两种不同的来源：
- 1.输入源（input source）
- 2.定时源（timer source）

两种源都使用程序的某一特定的处理例程来处理到达的事件。


当你创建输入源，你需要将其分配给run loop中的一个或多个模式。模式只会在特定事件影响监听的源。大多数情况下，run loop运行在默认模式下，但是你也可以使其运行在自定义模式。若某一源在当前模式下不被监听，那么任何其生成的消息只在run loop运行在其关联的模式下才会被传递。

![Runloop的结构和输入源类型](./assets/runloop.jpg) 

## 1.输入源（input source）

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
```objectc
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)argwaitUntilDone:(BOOL)wait modes:(NSArray *)array;
```
 

#### 2.定时源（timer source）

定时源在预设的时间点同步方式传递消息，这些消息都会发生在特定时间或者重复的时间间隔。定时源则直接传递消息给处理例程，不会立即退出run loop。

需要注意的是，尽管定时器可以产生基于时间的通知，但它并不是实时机制。和输入源一样，定时器也和你的run loop的特定模式相关。如果定时器所在的模式当前未被run loop监视，那么定时器将不会开始直到run loop运行在相应的模式下。类似的，如果定时器在run loop处理某一事件期间开始，定时器会一直等待直到下次run loop开始相应的处理程序。如果run loop不再运行，那定时器也将永远不启动。

