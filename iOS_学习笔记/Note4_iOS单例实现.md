# iOS 单例实现

> 参考资料1：[CSDN博客-moka](https://blog.csdn.net/m372897500/article/details/51332353)
> 参考资料2：[CSDN博客-Nicholas的专栏](https://blog.csdn.net/zhongbeida_xue/article/details/51438770)

iOS的单例模式有两种官方写法：（以下代码都是 [CSDN博客-moka](https://blog.csdn.net/m372897500/article/details/51332353)中的例子，以备后续查看）

__不使用GCD的方式__
实现代码如下：

```Objetive-C
#import "Manager.h"

static Manager *manager;

@implementation Manager

+ (Manager *)defaultManager {
  if (!manager) {
    manager = [[self allocWithZone:NULL] init];
  }

  return manager;
}
@end
```

__使用GCD的方式__
实现代码如下：

```Objetive-C
// iOS 单例的实现的写法（注明：dispatch_once这个函数，  它可以保证整个应用程序生命周期中某段代码只被执行一次！）

#import "Manager.h"

@implementation Manager

+ (Manager *) sharedManager {
  static dispatch_once_t predicate;
  static Manager *sharedManager;
  
  dispatch_once(&predicate, ^{
    sharedManager = [[Manager alloc] init];
  });
  return sharedManager;
}
@end
```

引出一个问题如下：

[从 Objective-C 里的 Alloc 和 AllocWithZone 谈起](http://www.cocoachina.com/bbs/read.php?tid=116873&fpage=33)

该主题作者博客地址： [博主：枫言枫语](http://justinyan.me/)

[Objective-C中单例模式的实现](http://cocoa.venj.me/blog/singleton-in-objc/)

[部分iOS示例代码](https://github.com/venj/Cocoa-blog-code)

> [简书 - iOS单例模式详解](https://www.jianshu.com/p/a54280d85c8b)
> [CSDN博客 - iOS单例模式详解](https://blog.csdn.net/huangfei711/article/details/53214411)