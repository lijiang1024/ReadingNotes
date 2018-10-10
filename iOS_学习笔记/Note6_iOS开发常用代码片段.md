# iOS开发常用代码片段

## 获取常用目录（iphone沙盒(sandbox)中的几个目录获取方式）

```Objective-C

// 获取沙盒主目录路径
NSString *homeDir = NSHomeDirectory();

// 获取Documents目录路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *docDir = [paths objectAtIndex:0];

// 获取Caches目录路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
NSString *cachesDir = [paths objectAtIndex:0];

// 获取tmp目录路径
NSString *tmpDir =  NSTemporaryDirectory();

// 获取当前程序包中一个图片资源（apple.png）路径
NSString *imagePath = [[NSBundle mainBundle] pathForResource:@"apple" ofType:@"png"];
UIImage *appleImage = [[UIImage alloc] initWithContentsOfFile:imagePath];

```