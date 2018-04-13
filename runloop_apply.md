# RunLoop应用
- NSTimer
- ImageView显示：控制方法在特定的模式下可用
- PerformSelector
- 常驻线程：在子线程中开启一个runloop
- AutoreleasePool 自动释放池
- UI更新

## 常驻线程

常驻线程的作用：我们知道，当子线程中的任务执行完毕之后就被销毁了，那么如果我们需要开启一个子线程，在程序运行过程中永远都存在，那么我们就会面临一个问题，如何让子线程永远活着，这时就要用到常驻线程：给子线程开启一个RunLoop
注意：子线程执行完操作之后就会立即释放，即使我们使用强引用引用子线程使子线程不被释放，也不能给子线程再次添加操作，或者再次开启。
子线程开启RunLoop的代码，先点击屏幕开启子线程并开启子线程RunLoop，然后点击button。
```objectivec
#import "ViewController.h"

@interface ViewController ()
@property(nonatomic,strong)NSThread *thread;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
}
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
   // 创建子线程并开启
    NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(show) object:nil];
    self.thread = thread;
    [thread start];
}
-(void)show
{
    // 注意：打印方法一定要在RunLoop创建开始运行之前，如果在RunLoop跑起来之后打印，RunLoop先运行起来，已经在跑圈了就出不来了，进入死循环也就无法执行后面的操作了。
    // 但是此时点击Button还是有操作的，因为Button是在RunLoop跑起来之后加入到子线程的，当Button加入到子线程RunLoop就会跑起来
    NSLog(@"%s",__func__);
    // 1.创建子线程相关的RunLoop，在子线程中创建即可，并且RunLoop中要至少有一个Timer 或 一个Source 保证RunLoop不会因为空转而退出，因此在创建的时候直接加入
    // 添加Source [NSMachPort port] 添加一个端口
    [[NSRunLoop currentRunLoop] addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    // 添加一个Timer
    NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(test) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];    
    //创建监听者
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
    CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);
    // 2.子线程需要开启RunLoop
    [[NSRunLoop currentRunLoop]run];
    CFRelease(observer);
}
- (IBAction)btnClick:(id)sender {
    [self performSelector:@selector(test) onThread:self.thread withObject:nil waitUntilDone:NO];
}
-(void)test
{
    NSLog(@"%@",[NSThread currentThread]);
}
@end
```
注意：创建子线程相关的RunLoop，在子线程中创建即可，并且RunLoop中要至少有一个Timer 或 一个Source 保证RunLoop不会因为空转而退出，因此在创建的时候直接加入，如果没有加入Timer或者Source，或者只加入一个监听者，运行程序会崩溃

作者：xx_cc
链接：https://www.jianshu.com/p/b9426458fcf6
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 自动释放池

* 作用

    * 自动释放对象的
    * 所有 `autorelease` 的对象，在出了作用域之后，会被自动添加到`最近创建的`自动释放池中
    * 自动释放池被销毁或者耗尽时，会向池中所有对象发送 `release` 消息，释放池中对象
    * 自动释放池，在 `ARC` & `MRC` 程序中，同样有效


![](./assets/自动释放池.jpg)

App启动后，苹果在主线程 RunLoop 里注册了两个 Observer管理和维护AutoreleasePool，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()，打印currentRunLoop可以看到。

```c
<CFRunLoopObserver 0x6080001246a0 [0x101f81df0]>{valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
<CFRunLoopObserver 0x608000124420 [0x101f81df0]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
```

- 第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

- 第二个 Observer 监视了两个事件，这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后： 
    - BeforeWaiting(准备进入休眠)时调用`_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()` 释放旧的池并创建新池；
    - Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。
       
主线程的其他操作通常均在这个AutoreleasePool之内（main函数中），以尽可能减少内存维护操作(当然你如果需要显式释放【例如循环】时可以自己创建AutoreleasePool否则一般不需要自己创建)。

### 常见面试题：

#### 1. 自动释放池是什么时候创建的？什么时候销毁的？

* 创建，运行循环检测到事件并启动后，就会创建自动释放池
* 销毁：一次完整的运行循环结束之前，会被销毁

#### 2. 以上代码是否有问题？如果有，如何解决？

```objectivec
for (long i = 0; i < largeNumber; ++i) {
    NSString *str = [NSString stringWithFormat:@"hello - %ld", i];
    str = [str uppercaseString];
    str = [str stringByAppendingString:@" - world"];
}
```
解决方法：引入自动释放池
* 1> 外面加自动释放池（快？）：能够保证for循环结束后，内部产生的自动释放对象，都会被销毁，需要等到 for 结束后，才会释放内存
* 2> 里面加自动释放池（慢？）：能够每一次 for 都释放产生的自动释放对象！
   
```objectivec
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {

    NSLog(@"start");
    CFAbsoluteTime start = CFAbsoluteTimeGetCurrent();
    [self answer1];
    NSLog(@"外 %f", CFAbsoluteTimeGetCurrent() - start);

    start = CFAbsoluteTimeGetCurrent();
    [self answer2];
    NSLog(@"内 %f", CFAbsoluteTimeGetCurrent() - start);
}

- (void)answer1 {
    @autoreleasepool {
        for (long i = 0; i < largeNumber; ++i) {
            NSString *str = [NSString stringWithFormat:@"hello - %ld", i];
            str = [str uppercaseString];
            str = [str stringByAppendingString:@" - world"];
        }
    }
}

- (void)answer2 {
    for (long i = 0; i < largeNumber; ++i) {
        @autoreleasepool {
            NSString *str = [NSString stringWithFormat:@"hello - %ld", i];
            str = [str uppercaseString];
            str = [str stringByAppendingString:@" - world"];
        }
    }
}
```

* **实际测试结果，是运行循环放在内部的速度更快！**
* 日常开发中，如果遇到局部代码内存峰值很高，可以引入运行循环及时释放延迟释放对象

## NSTimer


