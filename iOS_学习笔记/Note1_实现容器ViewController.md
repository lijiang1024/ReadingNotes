# 实现容器ViewController

## Implementing a Container View Controller

> 资料来源：[Implementing a Container View Controller](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html)

实现一个容器ViewController

### Designing a Custom Container View Controller

设计一个自定义的容器ViewController

**During the design process, ask yourself the following questions:**

在设计自定义容器ViewController时，要注意一下几点

* What is the role of the container and what role do its children play?

梳理清楚容器ViewController 和 子ViewController 的作用和角色

* How many children are displayed simultaneously?

要同时显示多少个 子ViewController？

* What is the relationship (if any) between sibling view controllers?

子ViewController 之间的关系是怎样的？

* How are child view controllers added to or removed from the container?

子ViewController 从 容器ViewController 中 添加 和 移除的时机

* Can the size or position of the children change? Under what conditions do those changes occur?

子ViewController 的 位置 和 大小是否会发生变化，触发 位置 和 大小 变化的条件是什么？

* Does the container provide any decorative or navigation-related views of its own?

容器ViewController 中是否有自己的 装饰View 或 导航相关的View

* What kind of communication is required between the container and its children? Does the container need to report specific events to its children other than the standard ones defined by the UIViewController class?

容器ViewController 和 子ViewController 之间的通讯方式是怎样的，容器ViewController 是否需要一些 UIViewController类提供的通用事件之外 特别的事件 和 子ViewController 通讯

* Can the appearance of the container be configured in different ways? If so, how?

容器ViewController 的外观是否可以配置，如果可以，怎样实现


iOS 系统提供的容器ViewController
* UINavigationController 

* UISplitViewController

* UITabBarController

-----
-----

### Configuring a Container in Interface Builder

**To create a parent-child container relationship at design time, add a container view object to your storyboard scene**

在storyboard视图设计器中使用 Container View 来关联 容器ViewController 和 子ViewController

-----
-----

### Implementing a Custom Container View Controller

-----

#### Adding a Child View Controller to Your Content

添加一个 子ViewController

To incorporate a child view controller into your content programmatically, create a parent-child relationship between the relevant view controllers by doing the following:

如果以编码的方式将 子ViewController 并入到 容器ViewController 中，执行下面的操作 在相关 ViewController 之间创建 父子关系：

* Call the addChildViewController: method of your container view controller.

This method tells UIKit that your container view controller is now managing the view of the child view controller.

* Add the child’s root view to your container’s view hierarchy.

Always remember to set the size and position of the child’s frame as part of this process.

* Add any constraints for managing the size and position of the child’s root view.

* Call the didMoveToParentViewController: method of the child view controller.

```Objective-C
- (void) displayContentController: (UIViewController*) content {
   [self addChildViewController:content];
   content.view.frame = [self frameForContentController];
   [self.view addSubview:self.currentClientView];
   [content didMoveToParentViewController:self];
}
```

In the preceding example, notice that you call only the didMoveToParentViewController: method of the child. That is because the addChildViewController: method calls the child’s willMoveToParentViewController: method for you. The reason that you must call the didMoveToParentViewController: method yourself is that the method cannot be called until after you embed the child’s view into your container’s view hierarchy.

在上面的例子中，请注意，只调用了 **didMoveToParentViewController:** 方法，这是因为 **addChildViewController:** 方法 会触发 子ViewController 的 **willMoveToParentViewController:** 方法调用；必须自己调用 **didMoveToParentViewController:** 方法是因为，在将 子ViewController 嵌入到 容器ViewController 层次结构中之前无法调用该方法。

When using Auto Layout, set up constraints between the container and child after adding the child to the container’s view hierarchy. Your constraints should affect the size and position of only the child’s root view. Do not alter the contents of the root view or any other views in the child’s view hierarchy.

在使用自动布局中，只需要设置 子ViewController 的 rootView 的约束。


#### Removing a Child View Controller

移除一个 子ViewController

To remove a child view controller from your content, remove the parent-child relationship between the view controllers by doing the following:

* Call the child’s willMoveToParentViewController: method with the value nil.

* Remove any constraints that you configured with the child’s root view.

* Remove the child’s root view from your container’s view hierarchy.

* Call the child’s removeFromParentViewController method to finalize the end of the parent-child relationship.


```Objective-C
- (void) hideContentController: (UIViewController*) content {
   [content willMoveToParentViewController:nil];
   [content.view removeFromSuperview];
   [content removeFromParentViewController];
}
```

Remove a child view controller from its container. Calling the willMoveToParentViewController: method with the value nil gives the child view controller an opportunity to prepare for the change. The removeFromParentViewController method also calls the child’s didMoveToParentViewController: method, passing that method a value of nil. Setting the parent view controller to nil finalizes the removal of the child’s view from your container.

-----

#### Transitioning Between Child View Controllers

子ViewController 之间的切换

```Objective-C
- (void)cycleFromViewController: (UIViewController*) oldVC
               toViewController: (UIViewController*) newVC {
   // Prepare the two view controllers for the change.
   [oldVC willMoveToParentViewController:nil];
   [self addChildViewController:newVC];
 
   // Get the start frame of the new view controller and the end frame
   // for the old view controller. Both rectangles are offscreen.
   newVC.view.frame = [self newViewStartFrame];
   CGRect endFrame = [self oldViewEndFrame];
 
   // Queue up the transition animation.
   [self transitionFromViewController: oldVC toViewController: newVC
        duration: 0.25 options:0
        animations:^{
            // Animate the views to their final positions.
            newVC.view.frame = oldVC.view.frame;
            oldVC.view.frame = endFrame;
        }
        completion:^(BOOL finished) {
           // Remove the old view controller and send the final
           // notification to the new view controller.
           [oldVC removeFromParentViewController];
           [newVC didMoveToParentViewController:self];
        }];
}
```

shows an example of how to swap one child view controller for another using a transition animation. In this example, the new view controller is animated to the rectangle currently occupied by the existing child view controller, which is moved offscreen. After the animations finish, the completion block removes the child view controller from the container. In this example, the transitionFromViewController:toViewController:duration:options:animations:completion: method automatically updates the container’s view hierarchy, so you do not need to add and remove the views yourself.


-----

#### Managing Appearance Updates for Children

管理 子ViewController 的外观


-----
-----

### Suggestions for Building a Container View Controller

Designing, developing, and testing a new container view controller takes time. Although the individual behaviors are straightforward, the controller as a whole can be quite complex. Consider the following tips when implementing your own container classes:

* **Access only the root view of a child view controller.** 
> A container should access only the root view of each child—that is, the view returned by the child’s view property. It should never access any of the child’s other views.

* **Child view controllers should have minimal knowledge of their container.**
>  A child view controller should focus on its own content. If the container allows its behavior to be influenced by a child, it should use the delegation design pattern to manage those interactions.

* **Design your container using regular views first.**
>  Using regular views (instead of the views from child view controllers) gives you an opportunity to test layout constraints and animated transitions in a simplified environment. When the regular views work as expected, swap them out for the views of your child view controllers.
