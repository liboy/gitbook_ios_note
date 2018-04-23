# Swift

## swift中的nil与OC中的nil区别

- OC中只有对象才能设置为nil，而swift中除了对象，Int、struct、enum等任何可选类型都可以等于nil
- OC中nil是一个指向不存在对象的指针。swift中，nil不是指针，nil是个确定的值，用来表示值缺失。

## delegate和block对比


4. 代理更注重过程信息的传输：比如发起一个网络请求，可能想要知道此时请求是否已经开始、是否收到了数据、数据是否已经接受完成、数据接收失败
block注重结果的传输：比如对于一个事件，只想知道成功或者失败，并不需要知道进行了多少或者额外的一些信息
5. Blocks 更清晰。比如 一个 viewController 中有多个弹窗事件，Delegate 就得对每个事件进行判断识别来源。而 Blocks 就可以在创建事件的时候区分开来了。这也是为什么现在苹果 API 中越来越多地使用 Blocks 而不是 Delegate。
6. block需要注意防止循环引用