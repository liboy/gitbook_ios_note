# Runloop
![](./assets/runloop.jpg)

## 概念
- 【Runloop 释义】："运行循环"、"跑圈"
- 【注解1】：iOS 中通常所说的 RunLoop 指的是 NSRunloop (Foundation框架) 或者 CFRunloopRef (CoreFoundation 框架) ，CFRunloopRef 是纯C的函数，而 NSRunloop 仅仅是 CFRunloopRef 的一层OC封装，并未提供额外的其他功能，因此要了解 RunLoop 内部结构，需要多研究 CFRunLoopRef API（Core Foundation \ 更底层）。
- 【注解2】：CFRunloopRef 其实就是 __CFRunloop 这个结构体指针（按照OC的思路我们可以将RunLoop看成一个对象），这个对象的运行才是我们通常意义上说的运行循环，核心方法是 __CFRunloopRun() 查看下（附：源码）。
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

【附】：CFRunLoop.c 源码
```c
# NOTE: 获得runloop实现 (创建runloop)

CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {// ✔️【主线程相关联的RunLoop创建】,如果为空，默认是主线程
        t = pthread_main_thread_np();
    }
    __CFLock(&loopsLock);
    if (!__CFRunLoops) { // 如果 RunLoop 不存在
        __CFUnlock(&loopsLock);
        // 创建字典
        CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        // 创建主线程对应的runloop
        CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        // 使用字典保存（KEY:线程 -- Value:线程对应的runloop）, 以保证一一对应关系。
        CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
        if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
            CFRelease(dict);
        }
        CFRelease(mainLoop);
        __CFLock(&loopsLock);
    }
    
    // ✔️【创建与子线程相关联的RunLoop】,从字典中获取 子线程的runloop
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
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```
【由上源码可得】：RunLoop 和 线程关系
1. 每条线程都有唯一的一个与之对应的RunLoop对象。
2. 主线程的RunLoop已经自动创建，子线程的RunLoop需要主动创建。
3. RunLoop在第一次获取时创建，在线程结束时销毁。

【注解】：Runloop 对象是利用字典来进行存储，而且 Key:线程 -- Value:线程对应的 runloop。
iOS开发过程中对于开发者而言更多的使用的是NSRunloop,它默认提供了三个常用的run方法


## 模式

* `RunLoop` 在同一段时间只能且必须在一种特定的模式下运行
* 如果要更换 Mode，必须先停止当前的 Loop，然后再重新启动 Loop
* Mode 是保证滚动流畅的关键

* `NSDefaultRunLoopMode`：默认状态、空闲状态
* `UITrackingRunLoopMode`：滚动模式
* `UIInitializationRunLoopMode`：私有的，App启动时
* `NSRunLoopCommonModes`：默认包含1，2两种模式

## 模拟 RunLoop 实现

```objc
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

```objc
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

```objc
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

```objc
- (IBAction)stop {
    if (_timerRf == NULL) {
        return;
    }

    CFRunLoopStop(_timerRf);
    _timerRf = NULL;
}
```