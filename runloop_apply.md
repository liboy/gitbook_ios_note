# RunLoop应用
- NSTimer
- ImageView显示：控制方法在特定的模式下可用
- PerformSelector
- 常驻线程：在子线程中开启一个runloop
- AutoreleasePool 自动释放池
- UI更新

## PerformSelecter

- performSelecter:afterDelay: 实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中
- performSelector:onThread: 实际上其会创建一个Timer加到对应的线程

## UI更新

当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。

苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数：
`_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()`函数遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

## GCD和RunLoop的关系

GCD并没有直接的关系。当调用`dispatch_async(dispatch_get_main_queue(), <#^(void)block#>)`时libDispatch会向主线程RunLoop发送消息唤醒RunLoop，RunLoop从消息中获取block，并且在`CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE`回调里执行这个block。不过仅限于主线程，其他线程dispatch操作是全部由libDispatch驱动的。

## 更多RunLoop使用

利用Observer对RunLoop进行监视
- 用perfromSelector在默认模式下设置图片，防止UITableView滚动卡顿
```
[[UIImageView alloc initWithFrame:CGRectMake(0, 0, 100, 100)] performSelector:@selector(setImage:) withObject:myImage afterDelay:0.0 inModes:@NSDefaultRunLoopMode]
```
- UITableView+FDTemplateLayoutCell利用Observer在界面空闲状态下计算出UITableViewCell的高度并进行缓存
- PerformanceMonitor关于iOS实时卡顿监控