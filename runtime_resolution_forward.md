# 方法解析与消息转发

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
    [message sendMessage:@"Sam Lau"];
}
``
控制台会打印以下信息：
```
normal way : send message = Sam Lau
```
但现在我将原来sendMessage方法实现给注释掉，覆盖resolveInstanceMethod方法：
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

method resolution way : send message = Sam Lau
注意到上面代码有这样一个字符串"v@*，它表示方法的参数和返回值，详情请参考Type Encodings。

如果resolveInstanceMethod方法返回NO，运行时就跳转到下一步：消息转发(Message Forwarding)