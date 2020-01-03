# iOS View 加载流程

> [参考资料：https://www.jianshu.com/p/b3b4f08e88c4](https://www.jianshu.com/p/b3b4f08e88c4)

控制器View的创建流程图

![控制器View的创建流程图](https://upload-images.jianshu.io/upload_images/1434508-370daf660ff330e4.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

控制器View的生命周期方法

```JavaScript
loadView：               加载view
viewDidLoad：            view加载完毕
viewWillAppear：         控制器的view将要显示
viewWillLayoutSubviews： 控制器的view将要布局子控件
viewDidLayoutSubviews：  控制器的view布局子控件完成
viewDidAppear:           控制器的view完全显示
viewWillDisappear：      控制器的view即将消失的时候
viewDidDisappear：       控制器的view完全消失的时候

// 作者：xx_cc
// 链接：https://www.jianshu.com/p/b3b4f08e88c4
// 來源：简书
// 简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```

view生命周期方法调用顺序

```JavaScript
viewDidLoad -> viewWillAppear -> viewWillLayoutSubviews -> viewDidLayoutSubviews -> viewDidAppear -> viewWillDisappear -> viewDidDisappear
```
