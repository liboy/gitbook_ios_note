# NSTimer

前面一直提到Timer Source作为事件源，事实上它的上层对应就是NSTimer（其实就是CFRunloopTimerRef）这个开发者经常用到的定时器（底层基于使用mk_timer实现），甚至很多开发者接触RunLoop还是从NSTimer开始的。其实NSTimer定时器的触发正是基于RunLoop运行的，所以使用NSTimer之前必须注册到RunLoop，但是RunLoop为了节省资源并不会在非常准确的时间点调用定时器，如果一个任务执行时间较长，那么当错过一个时间点后只能等到下一个时间点执行，并不会延后执行（NSTimer提供了一个tolerance属性用于设置宽容度，如果确实想要使用NSTimer并且希望尽可能的准确，则可以设置此属性）。

NSTimer的创建：
- 一种是timerWithXXX，


```objectivec
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
```
- scheduedTimerWithXXX。
```objectivec
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
```
区别：
就是后者除了创建一个定时器外会自动以NSDefaultRunLoopModeMode添加到当前线程RunLoop中
，不添加到RunLoop中的NSTimer是无法正常工作的。例如下面的代码中如果timer2不加入到RunLoop中是无法正常工作的。同时注意如果滚动UIScrollView（UITableView、UICollectionview是类似的）二者是无法正常工作的，但是如果将NSDefaultRunLoopMode改为NSRunLoopCommonModes则可以正常工作，这也解释了前面介绍的Mode内容。

    #import "ViewController1.h"

    @interface ViewController1 ()
    @property (nonatomic,weak) NSTimer *timer1;
    @property (nonatomic,weak) NSTimer *timer2;
    @end
    
    @implementation ViewController1
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        self.view.backgroundColor = [UIColor blueColor];
        // timer1创建后会自动以NSDefaultRunLoopMode默认模式添加到当前RunLoop中，所以可以正常工作
        self.timer1 = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timeInterval:) userInfo:nil repeats:YES];
        NSTimer *tempTimer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timeInterval:) userInfo:nil repeats:YES];
        // 如果不把timer2添加到RunLoop中是无法正常工作的(注意如果想要在滚动UIScrollView时timer2可以正常工作可以将NSDefaultRunLoopMode改为NSRunLoopCommonModes)
        [[NSRunLoop currentRunLoop] addTimer:tempTimer forMode:NSDefaultRunLoopMode];
        self.timer2 = tempTimer;
        
        CGRect rect = [UIScreen mainScreen].bounds;
        UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:CGRectInset(rect, 0, 200)];
        [self.view addSubview:scrollView];
        
        UIView *contentView = [[UIView alloc] initWithFrame:CGRectInset(scrollView.bounds, -100, -100)];
        contentView.backgroundColor = [UIColor redColor];
        [scrollView addSubview:contentView];
        scrollView.contentSize = contentView.frame.size;
    }
    
    - (void)timeInterval:(NSTimer *)timer {
        if (self.timer1 == timer) {
            NSLog(@"timer1...");
        } else {
            NSLog(@"timer2...");
        }
    }
    
    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self dismissViewControllerAnimated:true completion:nil];
    }
    
    - (void)dealloc {
        NSLog(@"ViewController1 dealloc...");
    }
    @end
注意上面代码中UIViewController1对timer1和timer2并没有强引用，对于普通的对象而言，执行完viewDidLoad方法之后（准确的说应该是执行完viewDidLoad方法后的的一个RunLoop运行结束）二者应该会被释放，但事实上二者并没有被释放。原因是：为了确保定时器正常运转，当加入到RunLoop以后系统会对NSTimer执行一次retain操作（特别注意：timer2创建时并没直接赋值给timer2，原因是timer2是weak属性，如果直接赋值给timer2会被立即释放，因为timerWithXXX方法创建的NSTimer默认并没有加入RunLoop，只有后面加入RunLoop以后才可以将引用指向timer2）。
但是即使使用了弱引用，上面的代码中ViewController1也无法正常释放，原因是在创建NSTimer2时指定了target为self，这样一来造成了timer1和timer2对ViewController1有一个强引用。解决这个问题的方法通常有两种：一种是将target分离出来独立成一个对象（在这个对象中创建NSTimer并将对象本身作为NSTimer的target），控制器通过这个对象间接使用NSTimer；另一种方式的思路仍然是转移target，只是可以直接增加NSTimer扩展（分类），让NSTimer自身做为target，同时可以将操作selector封装到block中。后者相对优雅，也是目前使用较多的方案（目前有大量类似的封装，例如：NSTimer+Block）。显然Apple也认识到了这个问题，如果你可以确保代码只在iOS 10下运行就可以使用iOS 10新增的系统级block方案（上面的代码中已经贴出这种方法）。
当然使用上面第二种方法可以解决控制器无法释放的问题，但是会发现即使控制器被释放了两个定时器仍然正常运行，要解决这个问题就需要调用NSTimer的invalidate方法（注意：无论是重复执行的定时器还是一次性的定时器只要调用invalidate方法则会变得无效，只是一次性的定时器执行完操作后会自动调用invalidate方法）。修改后的代码如下：

    #import "ViewController1.h"
    
    @interface ViewController1 ()
    @property (nonatomic,weak) NSTimer *timer1;
    @property (nonatomic,weak) NSTimer *timer2;
    @end
    
    @implementation ViewController1
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        self.view.backgroundColor = [UIColor blueColor];
    
        self.timer1 = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            NSLog(@"timer1...");
        }];
        NSTimer *tempTimer = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
            NSLog(@"timer2...");
        }];
        [[NSRunLoop currentRunLoop] addTimer:tempTimer forMode:NSDefaultRunLoopMode];
        self.timer2 = tempTimer;
        
        CGRect rect = [UIScreen mainScreen].bounds;
        UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:CGRectInset(rect, 0, 200)];
        [self.view addSubview:scrollView];
        
        UIView *contentView = [[UIView alloc] initWithFrame:CGRectInset(scrollView.bounds, -100, -100)];
        contentView.backgroundColor = [UIColor redColor];
        [scrollView addSubview:contentView];
        scrollView.contentSize = contentView.frame.size;
    }
    
    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self dismissViewControllerAnimated:true completion:nil];
    }
    
    - (void)dealloc {
        [self.timer1 invalidate];
        [self.timer2 invalidate];
        NSLog(@"ViewController1 dealloc...");
    }
    @end
其实和定时器相关的另一个问题大家也经常碰到，那就是NSTimer不是一种实时机制，官方文档明确说明在一个循环中如果RunLoop没有被识别（这个时间大概在50-100ms）或者说当前RunLoop在执行一个长的call out（例如执行某个循环操作）则NSTimer可能就会存在误差，RunLoop在下一次循环中继续检查并根据情况确定是否执行（NSTimer的执行时间总是固定在一定的时间间隔，例如1:00:00、1:00:01、1:00:02、1:00:05则跳过了第4、5次运行循环）。
要演示这个问题请看下面的例子（注意：有些示例中可能会让一个线程中启动一个定时器，再在主线程启动一个耗时任务来演示这个问，如果实际测试可能效果不会太明显，因为现在的iPhone都是多核运算的，这样一来这个问题会变得相对复杂，因此下面的例子选择在同一个RunLoop中即加入定时器和执行耗时任务）

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
如果运行并且不退出上面的程序会发现，前两秒NSTimer可以正常执行，但是两秒后由于同一个RunLoop中循环操作的执行造成定时器跳过了中间执行的机会一直到caculator循环完毕，这也正说明了NSTimer不是实时系统机制的原因。

但是以上程序还有几点需要说明一下：

NSTimer会对Target进行强引用直到任务结束或exit之后才会释放。如果上面的程序没有进行线程cancel而终止任务则及时关闭控制器也无法正确释放。
非主线程的RunLoop并不会自动运行（同时注意默认情况下非主线程的RunLoop并不会自动创建，直到第一次使用），RunLoop运行必须要在加入NSTimer或Source0、Sourc1、Observer输入后运行否则会直接退出。例如上面代码如果run放到NSTimer创建之前则既不会执行定时任务也不会执行循环运算。
performSelector:withObject:afterDelay:执行的本质还是通过创建一个NSTimer然后加入到当前线程RunLoop（通而过前后两次打印RunLoop信息可以看到此方法执行之后RunLoop的timer会增加1个。类似的还有performSelector:onThread:withObject:afterDelay:，只是它会在另一个线程的RunLoop中创建一个Timer），所以此方法事实上在任务执行完之前会对触发对象形成引用，任务执行完进行释放（例如上面会对ViewController形成引用，注意：performSelector: withObject:等方法则等同于直接调用，原理与此不同）。
同时上面的代码也充分说明了RunLoop是一个循环事实，run方法之后的代码不会立即执行，直到RunLoop退出。
上面程序的运行过程中如果突然dismiss，则程序的实际执行过程要分为两种情况考虑：如果循环任务caculate还没有开始则会在timer1中停止timer1运行（停止了线程中第一个任务），然后等待caculate执行并break（停止线程中第二个任务）后线程任务执行结束释放对控制器的引用；如果循环任务caculate执行过程中dismiss则caculate任务执行结束，等待timer1下个周期运行（因为当前线程的RunLoop并没有退出，timer1引用计数器并不为0）时检测到线程取消状态则执行invalidate方法（第二个任务也结束了），此时线程释放对于控制器的引用。
CADisplayLink是一个执行频率（fps）和屏幕刷新相同（可以修改preferredFramesPerSecond改变刷新频率）的定时器，它也需要加入到RunLoop才能执行。与NSTimer类似，CADisplayLink同样是基于CFRunloopTimerRef实现，底层使用mk_timer（可以比较加入到RunLoop前后RunLoop中timer的变化）。和NSTimer相比它精度更高（尽管NSTimer也可以修改精度），不过和NStimer类似的是如果遇到大任务它仍然存在丢帧现象。通常情况下CADisaplayLink用于构建帧动画，看起来相对更加流畅，而NSTimer则有更广泛的用处。




