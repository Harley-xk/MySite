---
title: UIStatusBarStyle 解惑
date: 2017-04-25 17:16
abstract: 作为一个强迫症晚期患者，对于 StatusBar 这样的细节也是无法放过的。对于每一个界面，StatusBar 都必须显示成需要的正确的风格，为此呕心沥血，殚精竭虑。。。
tags:
 - iOS
 - Swift
---

> 作为一个强迫症晚期患者，对于 StatusBar 这样的细节也是无法放过的。对于每一个界面，StatusBar 都必须显示成需要的正确的风格，为此呕心沥血，殚精竭虑。。。
> 而后有了这么一篇总结。

## 原始手段

要实现对 StatusBar Style 的绝对控制，其实有一个最简单粗暴的方法：

```swift
open func setStatusBarStyle(_ statusBarStyle: UIStatusBarStyle, animated: Bool)
```

这是从 iOS 2.0 那个古老的年代就已经存在的方法了，通过它可以直接设置 StatusBar 的 Style。

不过坏消息是，苹果一直在不遗余力地废弃老旧的 API，很不幸的，iOS 9 之后，这个方法也被标记为 **deprecated** 了：

```swift
@available(iOS, introduced: 2.0, deprecated: 9.0, message: "Use -[UIViewController preferredStatusBarStyle]")
open func setStatusBarStyle(_ statusBarStyle: UIStatusBarStyle, animated: Bool)
```

而且如果想要继续使用这个方式来管理 StatusBar Style 的话，还需要在 `Info.plist` 文件中设置：`View controller-based status bar appearance ` 为 `NO`。

对于一个强迫症晚期患者，代码里面满篇的 **deprecated** 警告以及 ` Info.plist ` 中额外的设置项是更加无法容忍的，这条路被无情地堵死了🙁

## 新的方向

其实 `setStatusBarStyle` 方法被废除也是意料之中的，如今编程思想已经在不知不觉中进步了不少，远古时代这种面向实现的编程思想已经与现如今的协议式编程相去甚远了。虽然老方法能够简单粗暴地解决问题，但是同样也留下了许多后遗症，比如在太过复杂的逻辑中，直接设置 StatusBar 的 Style 往往会造成混乱，最终得不到我们想要的效果；另外，由于我们可以在任何地方、任何代码逻辑中改变 StatusBar 的 Style，这对后期维护来说往往是灾难性的，最终导致得不偿失。

其实苹果早在 iOS 7 中就与时俱进地提供了新的控制 Status Bar Style 的体系，也就是上面的 `View controller-based status bar appearance`，通过限制改变 Status Bar Style 的自由性，转而交由 `View Controller` 去负责自己生命周期中的 Status Bar Style 控制。这么一来，整个思路都变得更佳清晰和优雅了。

### 基本方式

在新的体系下工作其实更简单了，只需要在 ViewController 中重写相关属性即可：

```swift
override var preferredStatusBarStyle: UIStatusBarStyle {
	return .lightContent
}
```

App 在运行过程中，始终由在 UI 最顶端的 ViewController 的 `preferredStatusBarStyle` 属性来决定当前 Status Bar 的 Style。当我们需要更改 Style 时，只需要在 `preferredStatusBarStyle` 计算方法中返回新的 `style` 属性值，然后调用 `setNeedsStatusBarAppearanceUpdate()` 方法即可：

```swift
override var preferredStatusBarStyle: UIStatusBarStyle {
	if needsLightContent {
		return .lightContent
	} else {
		return .default
	}
}
    
private func changeStyle() {
	needsLightContent = true
	setNeedsStatusBarAppearanceUpdate()
}
```

通过这种模式，我们始终通过 `ViewController` 来管理 Status Bar 的 Style，就算需要更新它的状态，也是通过间接的方式通知到 `ViewController` ，而不是直接改变 Status Bar 的属性。这样，后期的代码维护工作也可以更加轻松了。

### 分发控制权限

如果顶部的 `ViewController` 只是一个空壳，实际的 UI 逻辑都是由添加到之上的  `ChildController` 来控制的，那么这时候你可以通过重写 `childViewControllerForStatusBarStyle` 属性将 Status Bar Style 的控制权分发到对应的`ChildController` 中去：

```swift
override var childViewControllerForStatusBarStyle: UIViewController? {
	let currentChildViewController = childViewControllers[0]
	return currentChildViewController
}
```

### 特殊情况

当然，新的模式也引入了新的规则(keng)，我们在体会新的简单粗暴的手段时，也遇到了新的问题：如果不同的 `ViewController` 争夺控制权怎么办？比如 `Navigation` 和 `Modal Presentation` 这两个另类，他们都是一个 `ViewController` 嵌套在另一个 `ViewController` 的典型代表，这时候到底是谁说了算？

有坑就得填，首先拿 Navigation 开刀。

#### Navigation

当 UI 最顶层的 `ViewController` 嵌套在 UINavigationController 中时，UINavigationController 和 栈顶的 `ContentViewController` 都想争夺 Status Bar 的控制权，如果不加以控制，那刚刚建立的新秩序立刻就要被破坏了，于是我们有了下面的规则：

- 当 `Navigation Bar` 不可见时，由栈顶的 `ContentViewController` 负责管理 Status Bar 的 Style

- 当 `Navigation Bar` 可见时，由 `NavigationController` 负责管理 Status Bar 的 Style

  这时候，由于 `Navigation Bar` 处于可视状态，因此 Status Bar 需要与 `Navigation Bar` 的风格保持一致，因此由 `NavigationController` 来控制 Status Bar Style 是最合理的。

  但是这时候同样存在新的问题：`NavigationController` 往往都是使用系统默认的 `UINavigationController` 类，如果这时候的 Status Bar Style 不符合我们的需求怎么办？如果不继承 `UINavigationController` 就没法重写 `preferredStatusBarStyle` 属性；如果继承了 `UINavigationController` ，虽然可以实现需求，但是一来破坏了代码的简洁性，并且苹果是不建议继承 `UINavigationController` 的，二来有许多系统组件是直接继承自 `UINavigationController` 的（比如 `MFMailComposeViewController`），这时候继承也只是然并卵了。那么有什么办法可以解决这个问题呢？

必须有，苹果早就考虑到了这一点，于是针对 `UINavigationBar` 做了一番工作，提供了设置 Status Bar Style 的入口：`barStyle` 属性。`barStyle` 属性决定了 `NavigationBar` 的外观，因此，修改 `barStyle` 属性会联动改变 Status Bar 的 Style：

-   当 `barStyle` = `.default` 时，`NavigationBar` 显示为黑色，此时 `StatusBar`  显示为白色
  - 当 `barStyle` = `.black` 时，`NavigationBar` 显示为白色，此时 `StatusBar`  显示为黑色

  所以，如果通过设置 `barTintColor` 自定义了 `NavigationBar` 的颜色，这时候就需要设置 `barStyle` 属性来告知 `NavigationController` 正确的 Status Bar Style。

  *另外需要注意的是，由于某种原因，设置 `barStyle` 后会改变 `barTintColor`  的值，因此需要先修改 `barStyle` 属性，然后再设置 `barTintColor`。

#### Modal Presentation

在了解了 Navigation 之后，Model Presentation 的逻辑相对简单，通过 Modal Presentation 呈现的视图，如果也是嵌套在 `NavigationController` 中的话，此时的规则和 Navigaion 中描述的一致，否则的话同样由最顶端的 `ViewController` 来决定 Status Bar 的 Style。

需要注意的是，Modal Presentation 的逻辑有一个**前提条件**，只有当 `modalPresentationStyle` 的属性为 `.fullScreen` ，也就是全屏 Presentation 时，呈现出的视图才对 Status Bar Style 有发言权，否则还是由原先的 `ViewController` 来控制。

但是，凡事都有个但是不是么，如果你的强迫症已经无药可救了，非要在不全屏 Presentation 的 ViewController 中控制 Status Bar 的 Style，其实也不是不可以，`modalPresentationCapturesStatusBarAppearance` 是专门为强迫症患者量身定制的，只需要重写这个属性并返回 `true` ，你就可以在任何形式的 Model Presentation 中获得对 Status Bar Style 的控制权了。

## 结语

UIStatusBar 是大多数 App 都需要接触到的，但是往往也是最容易被忽视的一个细节，我们见过太多状态栏显示异常的 App (也包括某些大公司的产品)。很多时候这个代表了一个人、一个团队做事、做产品的态度。我一贯认为应该以对待一个艺术品而不是商品生产线的态度去开发软件，任何的细节都不应该放过。与其整天高谈阔论所谓的算法、性能，倒不如先从手头的细节做起，完善每一处用户体验，杜绝劣质 App。