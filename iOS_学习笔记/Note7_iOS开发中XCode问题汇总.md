# iOS开发XCode问题汇总

## 解决Xcode10 library not found for -lstdc++ 找不到问题

> [参考资料](https://www.jianshu.com/p/6d94278d62b3)

## Xcode provisioning profile 存储路径

> [参考资料地址](https://blog.csdn.net/zhouleizhao/article/details/46377173)

Xcode6 provisioning profile 存储路径：~/Library/MobileDevice/Provisioning Profiles

或者：Preferences->Accounts->ViewDetails可以浏览当前开发者账号上的信息，包括所有的Provisioning Profiles。

## ipa重签名

> [参考资料地址：](http://www.cocoachina.com/ios/20180530/23571.html?utm_source=tuicool&utm_medium=referral)

```Bash
# 重签名操作步骤
# 1、将ipa文件和mobileprovision文件放到同一目录下
# 2、将mobileprovision文件的名称修改为 embedded.mobileprovision
# 3、在终端中执行命令：fastlane sigh resign

fastlane sigh resign
```

## 查看静态库的架构

```Bash
lipo -info libCommon.a
```

## 常用宏变量

> [参考资料](http://www.cnblogs.com/ThankForYou/archive/2012/09/12/2681739.html)

宏|说明
|:-|:-|
__func__|打印当前函数或方法，c字符串
__LINE__|打印当前行号，整数
__FILE__|打印当前文件路径，c字符串
__PRETTY_FUNCTION__|打印当前函数或方法（在C++中会包含参数类型），c字符串

## 变量声明时修饰属性使用 copy assign weak strong

* Block 使用copy修饰
* Delegate 使用weak修饰

## iOS常量const正确使用

> [参考资料](https://www.cnblogs.com/mukekeheart/p/11395063.html)

我们需要确定的是以下的三种写法中前两种是一样的（可以修改kUserName的内容，也就是说const放在类型前还是类型后是一样的效果），第三种的效果不一样（无法修改kUserName的内容）。 

```OC
static NSString const * kUserName = @"StrongX";
static const NSString * kUserName = @"StrongX";

static NSString  * const kUserName = @"StrongX";
```

需要注意的是 **const 修饰的是他右边的部分**

```OC
static NSString const * kUserName = static NSString const (* kUserName )

static NSString  * const kUserName = static NSString  * const (kUserName)
```

当const修饰的是(userName)的时候，不可变的是userName。当const修饰的是( \* )的时候，“*”在C语言中表示指针指向符，也就是说这个时候userName指向的内存块地址不可变，而内存保存的内容是可变的，我们来做个尝试：

```OC
NSLog(@"内存地址： %x",& kUserName);
kUserName = @"superXLX";
NSLog(@"内存地址： %x",& kUserName);
```

所以当我们需要定义一个不可变的常量的时候 ，我们还是需要将“const”修饰符放到“*”指针指向符后边才对。

```OC
static NSString  * const kUserName = @"StrongX";
```
