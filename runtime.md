# 运行时(Runtime)机制

## 简介
Objective-C是基于C语言加入了面向对象特性和消息转发机制的动态语言，这意味着它不仅需要一个编译器，还需要Runtime系统来动态创建类和对象，进行消息发送和转发。下面通过分析Apple开源的[Runtime代码](https://opensource.apple.com/tarballs/objc4/)（我使用的版本是objc4-723.tar.gz）来深入理解Objective-C的Runtime机制。

### OC
* OC 是一个全动态语言，OC 的一切都是基于 Runtime 实现的
* 只有在程序运行时，才会去确定对象的类型，并调用类与对象相应的方法

### 运行时机制

* 运行时机制是用 C++ 开发的，是一套苹果开源的框架
* OC 是基于运行时开发的语言

### 应用场景

1. 运行时动态获取类的属性
    - 主要应用：字典转模型框架 `MJExtension`，`JSONModel`
2. 利用 `关联对象` 为分类添加属性
3. 利用 `交换方法` 拦截系统或其他框架的方法

> 误区：并不是使用的技术越底层，框架的效率就会越高


## 实战

* 导入头文件

```objectivec
#import <objc/runtime.h>
```

### 获取对象属性列表

#### 第1步－获取对象属性数量

```objectivec
+ (NSArray *)properties {

    unsigned int count = 0;
    class_copyPropertyList(self.class, &count);

    NSLog(@"属性数量 %u", count);

    return @[@"title", @"digest", @"replyCount", @"imgsrc"];
}
```

#### 第2步－获取对象属性名称

```objectivec
+ (NSArray *)properties {

    unsigned int count = 0;
    objc_property_t *list = class_copyPropertyList(self.class, &count);

    for (unsigned int i = 0; i < count; ++i) {
        const char *cname = property_getName(list[i]);
        printf("%s\t", cname);
    }
    printf("\n");

    // 释放属性数组
    free(list);

    return @[@"title", @"digest", @"replyCount", @"imgsrc"];
}
```

#### 第3步－生成属性数组

```objectivec
+ (NSArray *)properties {

    unsigned int count = 0;
    objc_property_t *list = class_copyPropertyList(self.class, &count);

    NSMutableArray *arrayM = [NSMutableArray arrayWithCapacity:count];
    for (unsigned int i = 0; i < count; ++i) {
        const char *cname = property_getName(list[i]);
        [arrayM addObject:[NSString stringWithUTF8String:cname]];
    }

    // 释放属性数组
    free(list);

    NSLog(@"%@", arrayM);

    return arrayM;
}
```

### 利用关联对象实现性能优化

```objectivec
///  关联对象键值
const char* propertiesKey = "propertiesKey";

...

// 判断是否能存在关联对象
NSMutableArray *arrayM = objc_getAssociatedObject(self, propertiesKey);
if (arrayM != nil) {
    return arrayM;
}

...

// 设置关联对象
objc_setAssociatedObject(self, propertiesKey, arrayM, OBJC_ASSOCIATION_COPY_NONATOMIC);
```
## 参考

https://www.jianshu.com/p/ab966e8a82e2
