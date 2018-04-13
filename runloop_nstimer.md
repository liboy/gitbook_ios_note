# NSTimer

前面一直提到Timer Source作为事件源，事实上它的上层对应就是NSTimer（其实就是CFRunloopTimerRef）这个开发者经常用到的定时器（底层基于使用mk_timer实现），甚至很多开发者接触RunLoop还是从NSTimer开始的。其实NSTimer定时器的触发正是基于RunLoop运行的，所以使用NSTimer之前必须注册到RunLoop，但是RunLoop为了节省资源并不会在非常准确的时间点调用定时器，如果一个任务执行时间较长，那么当错过一个时间点后只能等到下一个时间点执行，并不会延后执行（NSTimer提供了一个tolerance属性用于设置宽容度，如果确实想要使用NSTimer并且希望尽可能的准确，则可以设置此属性）。

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
### NSTimer可能存在误差

NSTimer不是一种实时机制
当前RunLoop在执行一个长的call out（例如执行某个循环操作），RunLoop在下一次循环中继续检查并根据情况确定是否执行

下面的例子选择在同一个RunLoop中即加入定时器和执行耗时任务
```
    #import "ViewController.h"
    
    @interface ViewController ()
    @property (nonatomic,weak) NSTimer *timer1;
    @property (nonatomic,strong) NSThread *thread1;
    @end
    
    @implementation ViewController
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        self.view.backgroundColor = [UIColor redColor];
        
        // 由于下面的方法无法拿到NSThread的引用，也就无法控制线程的状态
        //[NSThread detachNewThreadSelector:@selector(performTask) toTarget:self withObject:nil];
        self.thread1 = [[NSThread alloc] initWithTarget:self selector:@selector(performTask) object:nil];
        [self.thread1 start];
    }
    
    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self.thread1 cancel];
        [self dismissViewControllerAnimated:YES completion:nil];
    }
    
    - (void)dealloc {
        [self.timer1 invalidate];
        NSLog(@"ViewController dealloc.");
    }
    
    - (void)performTask {
        // 使用下面的方式创建定时器虽然会自动加入到当前线程的RunLoop中，但是除了主线程外其他线程的RunLoop默认是不会运行的，必须手动调用
        __weak typeof(self) weakSelf = self;
        self.timer1 = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            if ([NSThread currentThread].isCancelled) {
                //[NSObject cancelPreviousPerformRequestsWithTarget:weakSelf selector:@selector(caculate) object:nil];
                //[NSThread exit];
                [weakSelf.timer1 invalidate];
            }
            NSLog(@"timer1...");
        }];
        
        NSLog(@"runloop before performSelector:%@",[NSRunLoop currentRunLoop]);
        
        // 区分直接调用和「performSelector:withObject:afterDelay:」区别,下面的直接调用无论是否运行RunLoop一样可以执行，但是后者则不行。
        //[self caculate];
        [self performSelector:@selector(caculate) withObject:nil afterDelay:2.0];
    
        // 取消当前RunLoop中注册测selector（注意：只是当前RunLoop，所以也只能在当前RunLoop中取消）
        // [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(caculate) object:nil];
        NSLog(@"runloop after performSelector:%@",[NSRunLoop currentRunLoop]);
        
        // 非主线程RunLoop必须手动调用
        [[NSRunLoop currentRunLoop] run];
        
        NSLog(@"注意：如果RunLoop不退出（运行中），这里的代码并不会执行，RunLoop本身就是一个循环.");
        
        
    }
    
    - (void)caculate {
        for (int i = 0;i < 9999;++i) {
            NSLog(@"%i,%@",i,[NSThread currentThread]);
            if ([NSThread currentThread].isCancelled) {
                return;
            }
        }
    }
    
    @end
```
说明了NSTimer不是实时系统机制


CADisplayLink是一个执行频率（fps）和屏幕刷新相同（可以修改preferredFramesPerSecond改变刷新频率）的定时器，它也需要加入到RunLoop才能执行。与NSTimer类似，CADisplayLink同样是基于CFRunloopTimerRef实现，底层使用mk_timer（可以比较加入到RunLoop前后RunLoop中timer的变化）。和NSTimer相比它精度更高（尽管NSTimer也可以修改精度），不过和NStimer类似的是如果遇到大任务它仍然存在丢帧现象。通常情况下CADisaplayLink用于构建帧动画，看起来相对更加流畅，而NSTimer则有更广泛的用处。




