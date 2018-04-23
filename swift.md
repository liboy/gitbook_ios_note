# Swift

## swift中的nil与OC中的nil区别

- OC中只有对象才能设置为nil，而swift中除了对象，Int、struct、enum等任何可选类型都可以等于nil
- OC中nil是一个指向不存在对象的指针。swift中，nil不是指针，nil是个确定的值，用来表示值缺失。

## delegate和block对比

1. Delegate和Block一般都是一对一的通信；
2. Delegate需要定义协议方法，代理对象实现协议方法，并且需要建立代理关系才可以实现通信； Block更加简洁，不需要定义繁琐的协议方法，但通信事件比较多的话，建议使用Delegate；
3. delegate运行成本低。block成本很高的。
block出栈需要将使用的数据从栈内存拷贝到堆内存，当然对象的话就是加计数，使用完或者block置nil后才消除；delegate只是保存了一个对象指针，直接回调，没有额外消耗。相对C的函数指针，只多做了一个查表动作 .
