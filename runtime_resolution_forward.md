# 方法解析与消息转发

[receiver message]调用方法时，如果在message方法在receiver对象的类继承体系中没有找到方法，一般情况下，程序在运行时就会Crash掉，抛出`unrecognized selector sent to…`类似这样的异常信息。但在抛出异常之前，还有三次机会按以下顺序让你拯救程序。

- 方法解析（Method Resolution）
- 重定向（Fast Forwarding）
- 转发（Normal Forwarding）