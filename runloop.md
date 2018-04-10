# Runloop


## RunLoop对象
- CFRunLoopRef 是在 CoreFoundation 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。

- NSRunLoop 是基于 CFRunLoopRef 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

CFRunLoopRef 的代码是开源的，你可以在这里 http://opensource.apple.com/tarballs/CF 下载到整个 CoreFoundation 的源码。

### 获得RunLoop对象
```objectivec
//Foundation
[NSRunLoop currentRunLoop]; // 获得当前线程的RunLoop对象
[NSRunLoop mainRunLoop]; // 获得主线程的RunLoop对象

//Core Foundation
CFRunLoopGetCurrent(); // 获得当前线程的RunLoop对象
CFRunLoopGetMain(); // 获得主线程的RunLoop对象

//NSRunLoop <--> CFRunLoopRef 相互转化
NSLog(@"NSRunLoop <--> CFRunloop == %p--%p",CFRunLoopGetMain() , [NSRunLoop mainRunLoop].getCFRunLoop);

#【打印结果】：内存地址相同
0000-00-13 00:30:16.527 MultiThreading[57703:1217113] NSRunLoop <--> CFRunloop == 0x60000016a680--0x60000016a680
```

## 作用
1. 保持程序的持续运行（如：程序一启动就会开启一个主线程（中的 runloop 是自动创建并运行），runloop 保证主线程不会被销毁，也就保证了程序的持续运行）。
2. 处理App中的各种事件（如：touches 触摸事件、NSTimer 定时器事件、Selector事件（选择器 performSelector））。
3. 节省CPU资源，提高程序性能（有事情就做事情，没事情就休息 (其资源释放)）。
4. 负责渲染屏幕上的所有UI。

附：CFRunLoop.c 源码
```c
#【用DefaultMode启动，具体实现查看 CFRunLoopRunSpecific Line2704】
#【RunLoop的主函数，是一个死循环 dowhile】
void CFRunLoopRun(void) {   /* DOES CALLOUT */
    int32_t result;
    do {
        /*
         参数一：CFRunLoopRunSpecific   具体处理runloop的运行情况
         参数二：CFRunLoopGetCurrent()  当前runloop对象
         参数三：kCFRunLoopDefaultMode  runloop的运行模式的名称
         参数四：1.0e10                 runloop默认的运行时间，即超时为10的九次方
         参数五：returnAfterSourceHandled 回调处理
         */
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
        
        //【判断】：如果runloop没有停止 且 没有结束则继续循环，相反侧退出。
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}

#【直观表现】
RunLoop 其实内部就是do-while循环，在这个循环内部不断地处理各种任务（`比如Source、Timer、Observer`），
通过判断result的值实现的。所以 可以看成是一个死循环。
如果没有RunLoop，UIApplicationMain 函数执行完毕之后将直接返回，就是说程序一启动然后就结束；
```
## Runloop 开启&退出

验证 Runloop 是在UIApplicationMain 中开启。
```c

# int 类型返回值
UIKIT_EXTERN int UIApplicationMain(int argc, char *argv[], NSString * __nullable principalClassName, NSString * __nullable delegateClassName);

int main(int argc, char * argv[]) {
    @autoreleasepool {
        NSLog(@"开始");
        int number = UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
        NSLog(@"结束");
        return number;
    }
}

#【验证结果】：只会打印开始，并不会打印结束。
----
#【Runloop 的退出条件】。
App退出；线程关闭；设置最大时间到期；
```

>【注解】：说明在UIApplicationMain函数内部开启了一个和主线程相关的RunLoop (保证主线程不会被销毁)，导致 UIApplicationMain 不会返回，一直在运行中，也就保证了程序的持续运行。

## Runloop和线程关系
1. 每条线程都有唯一的一个与之对应的RunLoop对象，保存在一个字典里。
- 创建子线程RunLoop，通过`[NSRunLoop currentRunLoop]`在子线程内部获取，不获取则不会创建，方法调用时，会先查看字典，有则返回，没有则创建并存入字典中。
- 主线程的RunLoop已经自动创建，子线程的RunLoop需要主动创建。
- RunLoop在第一次获取时创建，在线程结束时销毁。

CFRunLoopRef源码
```c
# 1. 主线程相关联的RunLoop创建
   // 创建字典
    CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        // 创建主线程 根据传入的主线程创建主线程对应的RunLoop
    CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        // 保存主线程 将主线程-key和RunLoop-Value保存到字典中
    CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);

# 2. 创建与子线程相关联的RunLoop
    // 从字典中获取子线程的runloop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);
    if (!loop) {
        // 如果子线程的runloop不存在,那么就为该线程创建一个对应的runloop
    CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        // 把当前子线程和对应的runloop保存到字典中
    if (!loop) {
        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
        loop = newLoop;
    }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
    CFRelease(newLoop);
    }
```

## 模拟 RunLoop 实现

```objectc
#import <objc/message.h>

Person *person;

void callFunc(int type) {
    NSLog(@"正在执行 %d...%@", type, person);

    UInt32 result = ((UInt32 (*)(id, SEL, int, NSString *))objc_msgSend)(person, @selector(hahaha:name:), type, @"zhangsan");
    NSLog(@"耗时 %u 秒", result);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int result = 0;
        person = [[Person alloc] init];

        while (YES) {
            printf("请输入选择项，0表示退出：");
            scanf("%d", &result);

            if (result == 0) {
                printf("88\n");
                break;
            } else {
                callFunc(result);
            }
        }
    }
    return 0;
}
```

## 运行循环与时钟

```objectc
@interface ViewController ()
@property (nonatomic, strong) NSTimer *timer;
@end

@implementation ViewController

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(fire) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
}

- (void)fire {
    static int num = 0;

    ///  耗时操作
    for (int i = 0; i < 1000 * 1000; ++i) {
        [NSString stringWithFormat:@"hello - %d", i];
    }

    NSLog(@"%d", num++);
}

@end
```

> 运行测试，会发现卡顿非常严重

* 将时钟添加到其他线程工作

```objectc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(startTimer) object:nil];
    [self.thread start];
}

- (void)startTimer {
    @autoreleasepool {
        NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(fire) userInfo:nil repeats:YES];

        [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];

        _timerRf = CFRunLoopGetCurrent();
        CFRunLoopRun();

        NSLog(@"come here");
    }
}
```

> 注意：主线程的运行循环是默认启动的，但是子线程的运行循环是默认不工作的，这样能够保证线程执行完毕后，自动被销毁

* 停止运行循环

```objectc
- (IBAction)stop {
    if (_timerRf == NULL) {
        return;
    }

    CFRunLoopStop(_timerRf);
    _timerRf = NULL;
}
```


