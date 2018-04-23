# block

## 概念

* block 是 C 语言的
* 是一种数据类型，可以当作参数传递
* 是一组预先准备好的代码，在需要的时候执行

## block有3种类型：

- 全局块(_NSConcreteGlobalBlock)
    - 全局块存储在静态区（也叫全局区），相当于Objective-C中的单例
- 栈块(_NSConcreteStackBlock)
    - 栈块存储在栈区，超出作用域则马上被销毁
- 堆块(_NSConcreteMallocBlock)
    - 堆块存储在堆区中，是一个带引用计数的对象，需要自行管理其内存。

### block使用copy方法

| block的类型 | 存储区域 | 复制效果 |
| :-: | :-: | :-: |
|_NSConcreteStackBlock	|栈|从栈复制到堆|
|_NSConcreteGlobalBlock	|静态区(全局区)|什么也不做|
|_NSConcreteMallocBlock	|堆|引用计数增加|

copy修饰符的作用就是将block从栈区拷贝到堆区，主要目的就是保存block的状态，延长其生命周期。

## block中循环引用
方法一： 对Block内要使用的对象A使用_*_weak*进行修饰，Block对对象A弱引用打破循环。

有三种常用形式：

使用__weak ClassName
    __block XXViewController* weakSelf = self;
    self.blk = ^{
        NSLog(@"In Block : %@",weakSelf);
    };
使用__weak typeof(self)
    __weak typeof(self) weakSelf = self;
    self.blk = ^{
        NSLog(@"In Block : %@",weakSelf);
    };
Reactive Cocoa中的@weakify和@strongify
    @weakify(self);
    self.blk = ^{
        @strongify(self);
        NSLog(@"In Block : %@",self);
    };
其原理参考《@weakify, @strongify》，自己简便实现参考《@weak - @strong 宏的实现》

方法二：对Block内要使用的对象A使用__block进行修饰，并在代码块内，使用完__block变量后将其设为nil，并且该block必须至少执行一次。

    __block XXController *blkSelf = self;
    self.blk = ^{
        NSLog(@"In Block : %@",blkSelf);
    };
注意上述代码仍存在内存泄露，因为：

XXController对象持有Block对象blk
blk对象持有__block变量blkSelf
__block变量blkSelf持有XXController对象
    __block XXController *blkSelf = self;
    self.blk = ^{
        NSLog(@"In Block : %@",blkSelf);
        blkSelf = nil;//不能省略
    };
    
    self.blk();//该block必须执行一次，否则还是内存泄露
在block代码块内，使用完使用完__block变量后将其设为nil，并且该block必须至少执行一次后，不存在内存泄露，因为此时：

XXController对象持有Block对象blk
blk对象持有__block变量blkSelf(类型为编译器创建的结构体)
__block变量blkSelf在执行blk()之后被设置为nil（__block变量结构体的__forwarding指针指向了nil），不再持有XXController对象，打破循环
第二种使用__block打破循环的方法，优点是：

可通过__block变量动态控制持有XXController对象的时间，运行时决定是否将nil或其他变量赋值给__block变量
不能使用__weak的系统中，使用__unsafe_unretained来替代__weak打破循环可能有野指针问题，使用__block则可避免该问题
其缺点也明显：

必须手动保证__block变量最后设置为nil
block必须执行一次，否则__block不为nil循环应用仍存在
因此，还是避免使用第二种不常用方式，直接使用__weak打破Block循环引用。
方法三：将在Block内要使用到的对象（一般为self对象），以Block参数的形式传入，Block就不会捕获该对象，而将其作为参数使用，其生命周期系统的栈自动管理，不造成内存泄露。
即原来使用__weak的写法：

    __weak typeof(self) weakSelf = self;
    self.blk = ^{
        __strong typeof(self) strongSelf = weakSelf;
        NSLog(@"Use Property:%@", strongSelf.name);
        //……
    };
    self.blk();
改为Block传参写法后：

    self.blk = ^(UIViewController *vc) {
        NSLog(@"Use Property:%@", vc.name);
    };
    self.blk(self);
优点：

简化了两行代码，更优雅
更明确的API设计：告诉API使用者，该方法的Block直接使用传进来的参数对象，不会造成循环引用，不用调用者再使用weak避免循环


### 动画 block 回顾

```objectivec
self.demoView.center = CGPointMake(self.view.center.x, 0);
// 此方法会立即执行动画 block
[UIView animateWithDuration:2.0 delay:0 usingSpringWithDamping:0.3 initialSpringVelocity:10 options:0 animations:^{
    NSLog(@"动画开始");
    self.demoView.center = self.view.center;
} completion:^(BOOL finished) {
    // 会在动画结束后执行
    NSLog(@"动画完成");
}];
NSLog(@"come here");
```

## block 基本演练

* 最简单的 `block`

```objectivec
- (void)blockDemo1 {

    // 定义block
    // 类型 变量名 = 值
    void (^block)() = ^ {
        NSLog(@"Hello block");
    };

    // 执行
    block();
}
```

> 使用 `inlineBlock` 可以快速定义 `block`，不过 `block` 一定要过关

* 当作参数传递

```objectivec
- (void)blockDemo2 {
    void (^block)() = ^ {
        NSLog(@"Hello block");
    };

    [self demoBlock:block];
}

///  演示 block 当作参数传递
- (void)demoBlock:(void (^)())completion {
    NSLog(@"干点什么");

    completion();
}
```

* 使用局部变量

```objectivec
- (void)blockDemo3 {
    // 栈区变量
    int i = 10;
    NSLog(@"%p", &i);

    void (^block)() = ^ {
        // 定义 block 的时候会对栈区变量进行一次 copy
        NSLog(@"Hello block %d %p", i, &i);
    };

    [self demoBlock:block];
}
```

> 如果 block 中使用了外部变量，会对外部变量做一次 `copy`

* 在 block 中修改外部变量

```objectivec
- (void)blockDemo4 {
    // 栈区变量
    __block int i = 10;
    NSLog(@"%p", &i);

    void (^block)() = ^ {
        // 定义 block 的时候会对栈区变量进行一次 copy
        NSLog(@"Hello block %d %p", i, &i);
        i = 20;
    };

    NSLog(@"block 定义完成 %p %d", &i, i);

    [self demoBlock:block];

    NSLog(@"===>%d", i);
}
```

> 如果要在 block 内部修改栈区变量，需要使用 `__block` 修饰符，并且定义 block 之后，栈区变量的地址会变化为堆区地址

### block 的内存位置

* 全局区：如果block中没有使用任何全局变量
* 栈区：如果 block 中使用了外部变量
    * MRC 模式可以看到
    * ARC 模式，系统会自动将 Block 复制到堆中
* 堆区：将 block 设置给 copy 属性

```objectivec
@property (nonatomic, copy) void (^myBlock)();
```

```objectivec
- (void)blockDemo5 {
    int i = 10;
    void (^block)() = ^ {
        NSLog(@"i --- %d", i);
    };

    NSLog(@"%@", block);

    self.myBlock = block;
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {

    NSLog(@"%@", self.myBlock);
}
```

> 注意：虽然目前 ARC 编译器在设置属性时，已经替程序员复制了 block，但是定义 `block`时，仍然建议使用 `copy` 属性







