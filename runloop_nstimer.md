# NSTimer

NSTimer 其实就是 CFRunLoopTimerRef，他们之间是 toll-free bridged 的。一个 NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer 有个属性叫做 Tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差。如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

## 创建：
- 一种是timerWithXXX，

```objectivec
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
```
- 2. scheduedTimerWithXXX

```objectivec
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
```
### 区别：
- 后者除了创建一个定时器外会自动以NSDefaultRunLoopModeMode添加到当前线程RunLoop中
- 不添加到RunLoop中的NSTimer是无法正常工作的。

## NSTimer使用时的注意事项

- 注意timer添加到runloop时应该设置为什么mode
- 注意timer在不需要时，一定要调用invalidate方法释放定时器
- NSTimer不是一种实时机制，可能存在误差

## UITableView 与 NSTimer 冲突
解决方案：
```
1. 更改RunLoop运行Mode（NSRunLoopCommonModes）
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
2. 将NSTimer放到新的线程中
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
    [thread start];

- (void)newThread{
    @autoreleasepool{
        //在当前Run Loop中添加timer，模式是默认的NSDefaultRunLoopMode
        timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(incrementCounter:) userInfo: nil repeats:YES];
        //开始执行新线程的Run Loop，如果不启动run loop，timer的事件是不会响应的
        [[NSRunLoop currentRunLoop] run];
    }  
}
```


>CADisplayLink是一个执行频率（fps）和屏幕刷新相同（可以修改preferredFramesPerSecond改变刷新频率）的定时器，它也需要加入到RunLoop才能执行。与NSTimer类似，CADisplayLink同样是基于CFRunloopTimerRef实现，底层使用mk_timer（可以比较加入到RunLoop前后RunLoop中timer的变化）。和NSTimer相比它精度更高（尽管NSTimer也可以修改精度），不过和NStimer类似的是如果遇到大任务它仍然存在丢帧现象。通常情况下CADisaplayLink用于构建帧动画，看起来相对更加流畅，而NSTimer则有更广泛的用处。




