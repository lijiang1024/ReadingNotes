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
