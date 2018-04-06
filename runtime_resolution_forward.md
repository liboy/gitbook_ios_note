send messagesend messagesend messagesend message# 方法解析与消息转发

[receiver message]调用方法时，如果在message方法在receiver对象的类继承体系中没有找到方法，一般情况下，程序在运行时就会Crash掉，抛出`unrecognized selector sent to…`类似这样的异常信息。但在抛出异常之前，还有三次机会按以下顺序让你拯救程序。

1. 方法解析（Method Resolution）
- 重定向（Fast Forwarding）
- 转发（Normal Forwarding）

![](/assets/3.jpg)

## 1. 方法解析
首先Objective-C在运行时调用+ resolveInstanceMethod:或+ resolveClassMethod:方法，让你添加方法的实现。如果你添加方法并返回YES，那系统在运行时就会重新启动一次消息发送的过程。

举一个简单例子，定义一个类Message，它主要定义一个方法sendMessage，下面就是它的设计与实现：
``` objectc
@interface Message : NSObject
- (void)sendMessage:(NSString *)word;
@end
@implementation Message
- (void)sendMessage:(NSString *)word
{
    NSLog(@"normal way : send message = %@", word);
}
@end
```
在viewDidLoad方法调用sendMessage：
```objectc
- (void)viewDidLoad {
    [super viewDidLoad];
    Message *message = [Message new];
    [message sendMessage:@"send message"];
}
```
但现在我将原来sendMessage方法实现给注释掉，需要覆盖resolveInstanceMethod方法：
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

resolveInstanceMethod方法返回NO，就跳转到下一步：消息转发(Message Forwarding)

## 2.消息转发

### Fast Forwarding

如果目标对象实现- forwardingTargetForSelector:方法，系统就会在运行时调用这个方法，只要这个方法返回的不是nil或self，也会重启消息发送的过程，把这消息转发给其他对象来处理。否则，就会继续Normal Fowarding。

继续上面Message类的例子，将sendMessage和resolveInstanceMethod方法注释掉，然后添加forwardingTargetForSelector方法的实现：
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
此时还缺一个转发消息的类MessageForwarding，这个类的设计与实现如下：
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
此时，控制台会打印以下信息：

fast forwarding way : send message = Sam Lau
这里叫Fast，是因为这一步不会创建NSInvocation对象，但Normal Forwarding会创建它，所以相对于更快点。

### Normal Forwarding

如果没有使用Fast Forwarding来消息转发，最后只有使用Normal Forwarding来进行消息转发。它首先调用methodSignatureForSelector:方法来获取函数的参数和返回值，如果返回为nil，程序会Crash掉，并抛出unrecognized selector sent to instance异常信息。如果返回一个函数签名，系统就会创建一个NSInvocation对象并调用-forwardInvocation:方法。

继续前面的例子，将forwardingTargetForSelector方法注释掉，添加methodSignatureForSelector和forwardInvocation方法的实现：

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

