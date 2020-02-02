---
title: 实现 UITableViewCell 侧滑操作列表
date: 2017-04-07 14:27
abstract: 我们都知道 `UITableView` 支持实现侧滑操作，一般用来实现删除一个项目，实现起来也很简单，只需要实现 `UITableView` 的三个代理方法
tags:
 - iOS
 - Swift
---

## 曾经

我们都知道 `UITableView` 支持实现侧滑操作，一般用来实现删除一个项目，实现起来也很简单，只需要实现 `UITableView` 的三个代理方法：

- 首先告诉`UITableView`我们需要实现的操作类型，比如返回一个`.delete`

```swift
public func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCellEditingStyle {
    return .delete
}
```

- 然后告诉 UITableView 侧滑时删除按钮上显示的文字

```swift
func tableView(_ tableView: UITableView, titleForDeleteConfirmationButtonForRowAt indexPath: IndexPath) -> String? {
    return "Delete"
}
```

- 最后实现按钮触发后执行的操作

```swift
public func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
    // Do somthing
}
```

几个项目下来，我们对这个三部曲早就是滚瓜烂熟了。不过有时候会想，一个简单的操作按钮，需要实现三个 Delegate 方法来达到效果，会不会太繁琐了。。。

更何况还有一件更坑的事情：`editingStyleForRow`和`titleForDeleteConfirmationButton`这两个方法属于`UITableViewDelegate`协议，而`commit editingStyle`这个方法属于`UITableViewDataSource`协议。
这意味着，如果你为`UITableView`实现了通用的`DataSource`协议，那么要实现侧滑操作就不可避免要破坏代码结构了。。。

另外，侧滑支持多项操作的需求越来越旺盛，而此时我们的经典三部曲已经无法胜任工作了。

这时候要么选择第三方的实现方案，做好随时被坑的准备；要么，自己去实现一个更大的。。。坑？

## iOS 8 之后

估计苹果的工程师也为自己这个天才的设计折服了吧，于是在 iOS 8 引入了一个新的 API：`UITableViewRowAction`，先来看一看定义压压惊：

```swift
@available(iOS 8.0, *)
open class UITableViewRowAction : NSObject, NSCopying {

    public convenience init(style: UITableViewRowActionStyle, title: String?, handler: @escaping (UITableViewRowAction, IndexPath) -> Swift.Void)

    open var style: UITableViewRowActionStyle { get }

    open var title: String?

    @NSCopying open var backgroundColor: UIColor? // default background color is dependent on style

    @NSCopying open var backgroundEffect: UIVisualEffect?
}
```

先来看构造器，便利构造器接受三个属性：`style`、`title`、`handler`。

`title`这个不用说了，肯定就是按钮显示的标题了。

`style`通过定义可以看到，是一个`UITableViewRowActionStyle`类型的枚举值，通过构造器传入之后便不可更改了，想来是决定操作按钮的显示样式的吧，`destructive`这个单词是不是很熟悉？

```swift
@available(iOS 8.0, *)
public enum UITableViewRowActionStyle : Int {
    case `default`
    case destructive
    case normal
}
```

继续往下看到`handler`，这是一个闭包，显然是用来响应按钮点击事件的了，终于不用另外实现一个`Delegate`方法去响应操作事件了么？自从 iOS 4 之后引入了 block，苹果已经一发不可收拾了，API 改造大军正在路上。。。

class 定义里面还有两个属性：

首先看到`backgroundColor`，字面意思很好理解，就是背景颜色了，后面注释了一行小字：***default background color is dependent on style***。

这证实了我们的猜想：`style`属性决定了按钮的样式，也就是背景颜色，当然，我们也可以通过`backgroundColor`自己另外指定背景色。

最后一个属性是`backgroundEffect`，是`UIVisualEffect`类型的。`UIVisualEffect`？！这是啥？有经验的同学都知道，这是 iOS 7 之后引入的毛玻璃特效啊，不过研究了半天发现并没有卵用😂。也可能是我打开的方式不对？有知道的同学可以指点下。。。

## 实践

好了，看完了类定义，赶紧来看看怎么使用吧。先看一下`UITableViewDelegate`的定义，关于`UITableViewRowAction`只定义了一个方法：

```swift
func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath) -> [UITableViewRowAction]?
```

终于决定抛弃三部曲了么😳，返回值是个数组，这是支持多个操作项的节奏啊，废话少说上代码：

```swift
func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath) -> [UITableViewRowAction]? {
    let defaultAction = UITableViewRowAction(style: .default, title: "Default") 	{ (action, indexPath) in
        self.alertTitle("Default action at \(indexPath)")
    }
    let normalAction = UITableViewRowAction(style: .normal, title: "Normal") { (action, indexPath) in
        self.alertTitle("Normal action at \(indexPath)")
    }
    let destructiveAction = UITableViewRowAction(style: .destructive, title: "Delete") { (action, indexPath) in
        self.alertTitle("Delete action at \(indexPath)")
    }
    return [defaultAction, normalAction, destructiveAction]
}

func alertTitle(_ title: String) {
    let alert = UIAlertController(title: title, message: nil, preferredStyle: .alert)
    alert.addAction(UIAlertAction(title: "Cancel", style: .cancel))
    present(alert, animated: true)
}
```

这里定义了三个`UITableViewRowAction`，分别对应不同的`style`，运行之后可以发现，`default`和`destructive`默认都是红色，`normal`默认是灰色。这里只是简单定义了一个`alertTitle`方法用来响应点击反馈，实际项目中需要替换成特定的业务逻辑。

## 结语

至此终于水完了`UITableViewRowAction`相关的内容，虽然这是个 iOS 8 就出来的特性了，不过最近才被我注意到，而且还没有把`backgroundEffect`属性的特性摸清，实在是惭愧。

好消息是现在大多数 App 都是至少支持 iOS 8 + 了吧？可以再也不用写繁琐的三部曲了。
