# KVO

## 概念

* KVO - key value observer `键值`观察就是观察者模式。
 - 观察者模式的定义：一个 `目标对象` 管理所有依赖于它的 `观察者对象`，并在它自身的状态改变时主动通知`观察者对象`。这个`主动通知`通常是通过调用各观察者对象所提供的接口方法来实现的。`观察者模式`较完美地将 `目标对象` 与 `观察者对象` 解耦。
* `监听对象属性变化`的一种手段，可以用在开源框架，让代码解耦。例如：`上拉、下拉刷新控件`

## 底层实现
- `KVC` 和 `KVO` 都属于 `键值编程` 而且底层实现机制都是`isa-swizzing`

在KVO的中我们并不需要向`被观察者`添加额外的代码，就能在属性变化的时候得到通知，这个功能是如何实现的呢？同KVC一样依赖于强大的Runtime机制。

系统实现KVO有以下几个步骤：

- 当类A的对象第一次被观察的时候，系统会在运行期动态创建类A的派生类。我们称为B。
- 在派生类B中重写类A的`setter`方法，B类在被重写的setter方法中实现通知机制。
- 类B会重写 `class`方法，将自己伪装成类A。类B还会重写dealloc方法释放资源。
- 系统将所有指向类A对象的isa指针指向类B的对象。


isa指针的值并不一定反映实例的实际类，不能依靠isa指针来确定对象是否是一个类的成员。应该使用class方法来确定对象实例的类。

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
## 常见面试题
* NSNotification、KVO、Delegate 是同步的还是异步的？

### 结论
- NSNotification、KVO、Delegate在哪个线程中触发，就在哪个线程中响应，而且都是同步的，会阻塞当前线程，直到处理完成。
- 要注意避免阻塞主线程，如果存在耗时操作，建议在方法中先异步操作，再回到主线程做更新UI操作。

## 注意事项

* 监听方法执行会在属性变化所在的线程上执行！
* 如果多个线程同时修改一个属性，可能会出现资源抢夺的问题
* 如果监听的属性多，KVO 的监听方法会非常难写

> **对象销毁之前，一定要取消监听**
