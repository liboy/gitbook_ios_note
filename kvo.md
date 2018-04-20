# KVO

## 概念

* KVO - key value observer `键值`观察
* `监听对象属性变化`的一种手段，可以用在开源框架，让代码解耦。例如：`上拉、下拉刷新控件`

## 常见面试题

* NSNotification、KVO、Delegate 是同步的还是异步的？
  
  

## 代码演练

* 添加观察

```objectivec
// 添加键值观察
/**
 1. 调用对象：要监听的对象
 2. 参数
 1> 观察者，负责处理监听事件的对象
 2> 观察的属性
 3> 观察的选项
 4> 上下文
 */
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:@"Person Name"];
```

* 监听方法

```objectivec
// NSObject 分类方法，意味着所有的 NSObject 都可以实现这个方法！
// 跟协议的方法很像，分类方法又可以称为“隐式代理”！不提倡用，但是要知道概念！
// 所有的 kvo 监听到事件，都会调用此方法
/**
 1. 观察的属性
 2. 观察的对象
 3. change 属性变化字典（新/旧）
 4. 上下文，与监听的时候传递的一致

 可以利用上下文区分不同的监听！
 */
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {

    NSLog(@"睡会 %@", [NSThread currentThread]);

    [NSThread sleepForTimeInterval:1.0];

    NSLog(@"%@ %@ %@ %@", keyPath, object, change, context);
}
```

## 结论
- NSNotification、KVO、Delegate在哪个线程中触发，就在哪个线程中响应，而且都是同步的，会阻塞当前线程，直到处理完成。
- 要注意避免阻塞主线程，如果存在耗时操作，建议在方法中先异步操作，再回到主线程做更新UI操作。

## 注意事项

* 监听方法执行会在属性变化所在的线程上执行！
* 如果多个线程同时修改一个属性，可能会出现资源抢夺的问题
* 如果监听的属性多，KVO 的监听方法会非常难写

> **对象销毁之前，一定要取消监听**
