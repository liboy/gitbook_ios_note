# 方法解析与消息转发

抛出异常之前，还有三次机会按以下顺序让你拯救程序。

1. 方法解析（Method Resolution）
- 重定向（Fast Forwarding）
- 转发（Normal Forwarding）

![](/assets/3.png)

## 1. 方法解析
首先Objective-C在运行时调用`+ resolveInstanceMethod:`或`+ resolveClassMethod:`方法，如果你添加方法并返回YES，那系统在运行时就会重新启动一次消息发送的过程。

定义一个类Message：
``` objectc
@interface Message : NSObject
- (void)sendMessage:(NSString *)word;
@end

@implementation Message

- (void)viewDidLoad {
    [super viewDidLoad];
    Message *message = [Message new];
    [message sendMessage:@"send message"];
}

- (void)sendMessage:(NSString *)word {
    NSLog(@"normal way : send message = %@", word);
}

@end

```
注释掉sendMessage方法实现，覆盖resolveInstanceMethod方法：

```objectc
#pragma mark - Method Resolution
/// override resolveInstanceMethod or resolveClassMethod for changing sendMessage method implementation
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(sendMessage:)) {
        class_addMethod([self class], sel, imp_implementationWithBlock(^(id self, NSString *word) {
            NSLog(@"method resolution way : send message = %@", word);
        }), "v@*");
    }
    return YES;
}
```
控制台就会打印以下信息：
```
method resolution way : send message = send message
```
注意上面代码字符串"v@*，它表示方法的参数和返回值，详情请参考[Type Encodings](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)。

resolveInstanceMethod方法返回NO，就跳转到**消息转发(Message Forwarding)**

## 2.消息转发

### Fast Forwarding

如果目标对象实现`- forwardingTargetForSelector:`方法，系统就会在运行时调用这个方法，只要这个方法返回的不是nil或self，也会重启消息发送的过程，把这消息转发给其他对象来处理。否则，就会继续Normal Fowarding。

继续上面例子添加forwardingTargetForSelector方法的实现：
```objectc
#pragma mark - Fast Forwarding
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if (aSelector == @selector(sendMessage:)) {
        return [MessageForwarding new];
    }
    return nil;
}
```
类MessageForwarding实现如下：
```objectc
@interface MessageForwarding : NSObject
- (void)sendMessage:(NSString *)word;
@end
@implementation MessageForwarding
- (void)sendMessage:(NSString *)word
{
    NSLog(@"fast forwarding way : send message = %@", word);
}
@end
```

### Normal Forwarding

使用Normal Forwarding进行消息转发，首先调用methodSignatureForSelector:方法来获取函数的参数和返回值，如果返回为nil，程序会Crash掉，并抛出`unrecognized selector sent to instance`异常信息。如果返回一个函数签名，系统就会创建一个NSInvocation对象并调用-forwardInvocation:方法。

添加方法的实现如下：
```objectc
#pragma mark - Normal Forwarding
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    NSMethodSignature *methodSignature = [super methodSignatureForSelector:aSelector];
    if (!methodSignature) {
        methodSignature = [NSMethodSignature signatureWithObjCTypes:"v@:*"];
    }
    return methodSignature;
}
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    MessageForwarding *messageForwarding = [MessageForwarding new];
    if ([messageForwarding respondsToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:messageForwarding];
    }
}
```

forwardInvocation: 方法就是一个不能识别消息的分发中心，将这些不能识别的消息转发给不同的接收对象，或者转发给同一个对象，再或者将消息翻译成另外的消息，亦或者简单的“吃掉”某些消息，因此没有响应也不会报错。这一切都取决于方法的具体实现。

## 三种方法的选择

Runtime提供三种方式来将原来的方法实现代替掉，那该怎样选择它们呢？

- Method Resolution：由于Method Resolution不能像消息转发那样可以交给其他对象来处理，所以只适用于在原来的类中代替掉。
- Fast Forwarding：它可以将消息处理转发给其他对象，使用范围更广，不只是限于原来的对象。
- Normal Forwarding：它跟Fast Forwarding一样可以消息转发，但它能通过NSInvocation对象获取更多消息发送的信息，例如：target、selector、arguments和返回值等信息。

## 转发和多继承

- 转发和继承相似，可用于为 Objc 编程添加一些多继承的效果。
- 消息转发弥补了 Objc 不支持多继承的性质，也避免了因为多继承导致单个类变得臃肿复杂。
- 虽然转发可以实现继承的功能，但是 NSObject 还是必须表面上很严谨，像 respondsToSelector: 和 isKindOfClass: 这类方法只会考虑继承体系，不会考虑转发链。
