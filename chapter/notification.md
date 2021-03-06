# Notification（通知）

Notification对象封装了通知发送者想要传递给监听的的信息，它有3个属性：
```objectivec
@property (readonly, copy) NSString *name;  // 通知的标识名称(一般为常量字符串)
@property (readonly, retain) id object;  // 任意想要携带的对象，通常为发送者自己
@property (readonly, copy) NSDictionary *userInfo; // 关于通知的附加信息
 
```

通知就是以Notification的形式从通知发送者发出，到通知中心，然后再分发给所有监听该通知的对象的，通知监听者们接收到通知之后，可以获取到传递过来的Notification对象，从而获取里面封装的一些信息，做相应的处理

## 优势

1. 不需要编写多少代码，实现比较简单；
2. 对于一个发出的通知，多个对象能够做出反应，即1对多的方式实现简单
3. 能够传递context对象（dictionary），context对象携带了关于发送通知的自定义的信息

## 缺点：

1. 在编译期不会检查通知是否能够被观察者正确的处理；
2. 在释放注册的对象时，需要在通知中心取消注册；
3. 在调试的时候应用的工作以及控制过程难跟踪；
4. 需要第三方对喜爱那个来管理controller与观察者对象之间的联系；
5. 需要提前知道通知名称、UserInfo dictionary keys。如果这些没有在工作区间定义，那么会出现不同步的情况；
6. 通知发出后，controller不能从观察者获得任何的反馈信息。

## NSNotificationCenter（通知中心）

通知中心是整个通知机制的关键所在，它管理着监听者的注册和注销，通知的发送和接收。通知中心维护着一个通知的分发表，把所有发送者发送的通知，转发给对应的监听者们。每一个iOS程序都有一个唯一的通知中心，你不必自己去创建一个，它是一个单例，通过 `[NSNotificationCenter defaultCenter]` 方法获取。

### 注册监听者方法:
```objectivec
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullable NSString *)aName object:(nullable id)anObject;
- (id <NSObject>)addObserverForName:(nullable NSString *)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block;
```
第一个方法是大家常用的方法，不用多说，第二个方法带了一个block，这个block就是通知被触发时要执行的block，这个block带有一个notification参数；该方法还有一个queue参数，可以指定这个block在哪个队列上执行，如果传nil，这个block将会在发送通知的线程中同步执行。然后注意到，这个方法有一个id类型的返回值，这个返回值是一个不透明的中间值，用来充当监听者，使用时，我们需要将这个返回的监听者保存起来，在后面移除监听者的时候用到。

### 移除监听者方法:
```objectivec
- (void)removeObserver:(id)observer;
- (void)removeObserver:(id)observer name:(nullable NSString *)aName object:(nullable id)anObject;
```
在监听对象销把该对象监听的通知移除掉。

### 发送通知方法:
```objectivec
- (void)postNotification:(NSNotification *)notification;
- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject;
- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject userInfo:(nullable NSDictionary *)aUserInfo;
```

- 通知中心默认是以同步的方式发送通知的，也就是说，当一个对象发送了一个通知，只有当该通知的所有接受者都接受到了通知中心分发的通知消息并且处理完成后，发送通知的对象才能继续执行接下来的方法。异步发送通知的方法下面会说到。
- 在一个多线程的程序中，发送方发送通知的线程通常就是监听者接受通知的线程，这可能和监听者注册时的线程不一样。

## NSNotificationQueue（通知队列）

- 通知队列（Notification queue）更像是通知中心的缓冲区。
- 通知队列通常以先进先出（FIFO）的顺序管理通知。
- 当一个通知上升到队列的前面时，队列就将它发送给通知中心，通知中心随后将它派发给所有注册为观察者的对象。
- 每个线程有一个默认的通知队列，它和通知中心关联着，你也可以自己为线程或者通知中心创建多个通知队列。

![NotificationQueue](/assets/notification1.png)

通知队列给通知机制提供了2个重要的特性：
- 通知合并
- 异步发送通知

### 通知合并

把和刚进入队列的通知相类似的其它通知从队列中移除的过程。如果一个新的通知和已经在队列中的通知相类似，则新的通知不进入队列，而所有类似的通知（除了队列中的第一个通知以外）都被移除。但是，为安全起见，开发者不应该完全依赖于这个特殊的合并行为。

coalesceMask有3个给定的值：
```objectivec
typedef NS_OPTIONS(NSUInteger, NSNotificationCoalescing) {
    //不合并
    NSNotificationNoCoalescing = 0,
    //按通知的名字合并
    NSNotificationCoalescingOnName = 1,
    //按通知的发送者合并
    NSNotificationCoalescingOnSender = 2
};
```
合并规则还可以用|符号连接，指定多个
```objectivec
NSNotification *myNotification = [NSNotification notificationWithName:@"MyNotificationName" object:nil];
[[NSNotificationQueue defaultQueue] enqueueNotification:myNotification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationCoalescingOnName | NSNotificationCoalescingOnSender forModes:nil];
```


### 异步发送通知
使用通知队列的下面2个方法，将通知加到通知队列中，就可以将一个通知异步的发送到当前的线程，这些方法调用后会立即返回，不用再等待通知的所有监听者都接收并处理完。
```objectivec
- (void)enqueueNotification:(NSNotification *)notification postingStyle:(NSPostingStyle)postingStyle;
- (void)enqueueNotification:(NSNotification *)notification postingStyle:(NSPostingStyle)postingStyle coalesceMask:(NSNotificationCoalescing)coalesceMask forModes:(nullable NSArray<NSString *> *)modes;
```
>注意：如果通知入队的线程在该通知被通知队列发送到通知中心之前结束了，那么这个通知将不会被发送了。

- postingStyle参数
    - NSPostASAP （尽快发送 Posting As Soon As Possible）
    - NSPostWhenIdle（空闲时发送）
    - NSPostNow（立即发送）
- modes参数
    - 某种特定runloop mode后，该通知值有在当前runloop为指定mode的下，才会被发出







