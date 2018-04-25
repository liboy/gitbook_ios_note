# Notification（通知）

Notification对象封装了通知发送者想要传递给监听的的信息，它有3个属性：

```objectivec
@property (readonly, copy) NSString *name;
@property (nullable, readonly, retain) id object;
@property (nullable, readonly, copy) NSDictionary 
*userInfo;
```
name：通知的名称，用来标示一个通知，一般为字符串
object：任意想要携带的对象，通常为发送者自己
userInfo：附加信息

通知就是以Notification的形式从通知发送者发出，到通知中心，然后再分发给所有监听该通知的对象的，通知监听者们接收到通知之后，可以获取到传递过来的Notification对象，从而获取里面封装的一些信息，做相应的处理

作者：我叫纠结伦
链接：https://www.jianshu.com/p/8832f019c17f
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
在IOS应用开发中有一个”Notification Center“的概念。它是一个单例对象，允许当事件发生时通知一些对象。它允许我们在低程度耦合的情况下，满足控制器与一个任意的对象进行通信的目的。这种模式的基本特征是为了让其他的对象能够接收到在该controller中发生某种事件而产生的消息，controller用一个key（通知名称）。这样对于controller来说是匿名的，其他的使用同样的key来注册了该通知的对象（即观察者）能够对通知的事件作出反应。

## 优势

1. 不需要编写多少代码，实现比较简单；

2. 对于一个发出的通知，多个对象能够做出反应，即1对多的方式实现简单

3. 能够传递context对象（dictionary），context对象携带了关于发送通知的自定义的信息

## 缺点：

1. 在编译期不会检查通知是否能够被观察者正确的处理；

2. 在释放注册的对象时，需要在通知中心取消注册；

3. 在调试的时候应用的工作以及控制过程难跟踪；

4. 需要第三方对喜爱那个来管理controller与观察者对象之间的联系；

5. 需要提前知道通知名称、UserInfo dictionary keys。如果这些没有在工作区间定义，那么会出现不同步的情况；

6. 通知发出后，controller不能从观察者获得任何的反馈信息。
