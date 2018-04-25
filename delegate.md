# Delegate

delegation的基本特征是，一个controller定义了一个协议（即一系列的方法定义）。该协议描述了一个delegate对象为了能够响应一个controller的事件而必须做的事情。协议就是delegator说，“如果你想作为我的delegate，那么你就必须实现这些方法”。实现这些方法就是允许controller在它的delegate能够调用这些方法，而它的delegate知道什么时候调用哪种方法。delegate可以是任何一种对象类型，因此controller不会与某种对象进行耦合，但是当该对象尝试告诉委托事情时，该对象能确定delegate将响应。

## 优点

1. 非常严格的语法。所有将听到的事件必须是在delegate协议中有清晰的定义。
2. 如果delegate中的一个方法没有实现那么就会出现编译警告/错误
3. 协议必须在controller的作用域范围内定义
4. 在一个应用中的控制流程是可跟踪的并且是可识别的；
5. 在一个控制器中可以定义定义多个不同的协议，每个协议有不同的delegates

6. 没有第三方对象要求保持/监视通信过程。

7. 能够接收调用的协议方法的返回值。这意味着delegate能够提供反馈信息给controller

## 缺点：

1. 需要定义很多代码：1.协议定义；2.controller的delegate属性；3.在delegate本身中实现delegate方法定义

2. 在释放代理对象时，需要小心的将delegate改为nil。一旦设定失败，那么调用释放对象的方法将会出现内存crash

3. 在一个controller中有多个delegate对象，并且delegate是遵守同一个协议，但还是很难告诉多个对象同一个事件，不过有可能。


