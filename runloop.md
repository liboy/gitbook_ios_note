# 运行循环

## 作用

* **保持程序持续运行**，程序一启动就会开一个主线程，主线程一开起来就会跑一个主线程对应的RunLoop,RunLoop保证主线程不会被销毁，也就保证了程序的持续运行
* 决定程序在何时处理哪些事件
* 调用解耦（Message Queue）
* 节省CPU时间（当程序启动后，什么都没有执行的话，就不用让CPU来消耗资源来执行，直接进入睡眠状态）

![](./assets/runloop.jpg)

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