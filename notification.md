# Notification（通知）

Notification对象封装了通知发送者想要传递给监听的的信息，它有3个属性：
```objectivec
@property (readonly, copy) NSString *name;
@property (nullable, readonly, retain) id object;
@property (nullable, readonly, copy) NSDictionary 
*userInfo;
```
- name：通知的名称，用来标示一个通知，一般为字符串
- object：任意想要携带的对象，通常为发送者自己
- userInfo：附加信息

通知就是以Notification的形式从通知发送者发出，到通知中心，然后再分发给所有监听该通知的对象的，通知监听者们接收到通知之后，可以获取到传递过来的Notification对象，从而获取里面封装的一些信息，做相应的处理



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
