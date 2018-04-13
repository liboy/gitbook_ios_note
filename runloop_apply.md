# RunLoop应用
- NSTimer
- ImageView显示：控制方法在特定的模式下可用
- PerformSelector
- 常驻线程：在子线程中开启一个runloop
- AutoreleasePool 自动释放池
- UI更新

## PerformSelecter

当调用 NSObject 的 performSelecter:afterDelay: 后，实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。所以如果当前线程没有 RunLoop，则这个方法会失效。

当调用 performSelector:onThread: 时，实际上其会创建一个 Timer 加到对应的线程去，同样的，如果对应线程没有 RunLoop 该方法也会失效。

## UI更新

如果打印App启动之后的主线程RunLoop可以发现另外一个callout为_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv 的 Observer，这个监听专门负责UI变化后的更新，比如修改了frame、调整了UI层级（UIView/CALayer）或者手动设置了setNeedsDisplay/setNeedsLayout 之后就会将这些操作提交到全局容器。而这个Observer监听了主线程RunLoop的即将进入休眠和退出状态，一旦进入这两种状态则会遍历所有的UI更新并提交进行实际绘制更新。

通常情况下这种方式是完美的，因为除了系统的更新，还可以利用 setNeedsDisplay 等方法手动触发下一次 RunLoop 运行的更新。但是如果当前正在执行大量的逻辑运算可能UI的更新就会比较卡，因此facebook 推出了 AsyncDisplayKit 来解决这个问题。AsyncDisplayKit 其实是将UI排版和绘制运算尽可能放到后台，将UI的最终更新操作放到主线程（这一步也必须在主线程完成），同时提供一套类 UIView 或 CALayer 的相关属性，尽可能保证开发者的开发习惯。这个过程中 AsyncDisplayKit 在主线程 RunLoop 中增加了一个Observer 监听即将进入休眠和退出 RunLoop 两种状态,收到回调时遍历队列中的待处理任务一一执行。

## GCD和RunLoop的关系

在RunLoop的源代码中可以看到用到了GCD的相关内容，但是RunLoop本身和GCD并没有直接的关系。当调用了dispatch_async(dispatch_get_main_queue(), <#^(void)block#>)时libDispatch会向主线程RunLoop发送消息唤醒RunLoop，RunLoop从消息中获取block，并且在__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__回调里执行这个block。不过这个操作仅限于主线程，其他线程dispatch操作是全部由libDispatch驱动的。

## 更多RunLoop使用

利用Observer对RunLoop进行监视
- 用perfromSelector在默认模式下设置图片，防止UITableView滚动卡顿
```
[[UIImageView alloc initWithFrame:CGRectMake(0, 0, 100, 100)] performSelector:@selector(setImage:) withObject:myImage afterDelay:0.0 inModes:@NSDefaultRunLoopMode]
```
- UITableView+FDTemplateLayoutCell利用Observer在界面空闲状态下计算出UITableViewCell的高度并进行缓存
- PerformanceMonitor关于iOS实时卡顿监控