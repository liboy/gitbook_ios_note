# Delegate

>代理是一种简单而功能强大的设计模式，这种模式用于一个对象“代表”另外一个对象和程序中其他的对象进行交互。 主对象（这里指的是delegating object）中维护一个代理（delegate）的引用并且在合适的时候向这个代理发送消息。这个消息通知“代理”主对象即将处理或是已经处理完了某一个事件。这个代理可以通过更新自己或是其它对象的UI界面或是其它状态来响应主对象所发送过来的这个事件的消息。或是在某些情况下能返回一个值来影响其它即将发生的事件该如何来处理。代理的主要价值是它可以让你容易的定制各种对象的行为。注意这里的代理是个名词，它本身是一个对象，这个对象是专门代表被代理对象来和程序中其他对象打交道的。

协议和代理是模块化开发和封装的产物。

![delegate](/assets/delegate.png)

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

3. 可以有多个delegate对象，并且delegate是遵守同一个协议，但还是很难告诉多个对象同一个事件，不过有可能。


