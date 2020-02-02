---
title: Swift 造轮子之自制 HUD
date: 2017-11-13 17:16
abstract: HUD 是每个 iOS App 的必备组件之一，市面上 Objective-C 的版本也是五花八门。然而转入 Swift 之后并没有一个明星组件出现，仅有的那几个也不能完全满足需求，无奈只能自己造轮子了
tags:
 - iOS
 - Swift
---

> HUD 是每个 iOS App 的必备组件之一，市面上 Objective-C 的版本也是五花八门。然而转入 Swift 之后并没有一个明星组件出现，仅有的那几个也不能完全满足需求，无奈只能自己造轮子了。

## Chrysan

Chrysan 取自单词 chrysanthemum 的前半部分，意为菊花，也是 HUD 的通俗称呼。所以就这么叫着吧～

其实 Chrysan 这个库写完已经有一段时间了，经过了好几个线上项目的考验，暂时没有发现什么重大问题，所以最近把它推送到了 CocoaPods，打算分享出来造福大众，顺便骗一波 star 😂

## 安装

Chrysan 已经推送到 pod repo，所以集成非常简单：

```ruby
pod 'Chrysan'
```

当然，也可以在我的 [Github](https://github.com/Harley-xk/) 上直接下载源代码：https://github.com/Harley-xk/Chrysan

## 使用

Chrysan 设计为每个 View 都可以创建出 HUD，实践证明这个特性在某些特殊情况下非常有用。每个 View 都有一个 chrysan 属性，通过它可以创建当前 View 的独立的菊花。只有当第一次访问 chrysan 属性时才会真实地创建 ChrysanView 实例，避免不必要的开销和内存占用。

通过访问 ViewController 的 chrysan 属性，可以访问 ViewController 的根 View 的菊花并自动创建。

### 显示

```swift
public func show(_ status: Status = .running, message: String? = nil, hideDelay delay: Double = 0)
```

这个方法用来显示一个菊花，各参数说明如下：

***message*** - 显示在图标下方的说明文字，说明文字支持多行文本

***hideDelay*** - 自动隐藏的时间，传入0则表示不自动隐藏

***status*** - 显示菊花的状态，属于 Status 枚举类型，可以控制显示不同的状态

```swift
/// 菊花的状态，不同的状态显示不同的icon
    public enum Status {
        /// 无状态，显示纯文字
        case plain
        /// 执行中，显示菊花
        case running
        /// 进度，环形进度条
        case progress
        /// 成功，显示勾
        case succeed
        /// 错误，显示叉
        case error
        /// 自定义，显示自定义的 icon
        case custom
    }
```

### 显示菊花

由于 show 方法的各个参数都支持默认值，因此可以调用所有参数都是默认值的最简形式，此时显示一个单纯的不会隐藏的菊花。

```swift
chrysan.show()
```

如果需要显示带文字的菊花，只需要简单加上一个 `message` 参数就可以了：

``` swift
chrysan.show(message: "正在处理")
```

### 显示纯文本

```swift
// 显示纯文字
chrysan.show(.plain, message:"这是一段纯文字")
// 显示纯文字，1 秒后隐藏
chrysan.show(.plain, message:"这是一段纯文字", hideDelay: 1)
```

### 显示图案

```swift
// 任务完成后给予用户反馈
chrysan.show(.succeed, message: "处理完毕", hideDelay: 1)
// 显示自定义图案
let image = UIImage(named: "myImage")
chrysan.show(customIcon: image, message: "自定义图案", hideDelay: 1)
```

### 显示任务进度

```swift
// 显示环形的任务进度，会在中心显示进度百分比，progress 取值 0-1
chrysan.show(progress: progress, message: "下载中...")
```

## 自定义样式

Chrysan 支持有限的自定义样式

菊花背景支持 UIBlurEffect 的所有样式

```swift
/// 菊花背景样式，使用系统自带的毛玻璃特效，默认为黑色样式
public var hudStyle = UIBlurEffectStyle.dark
```

菊花使用系统的 UIActivityIndicatorView，支持 UIActivityIndicatorViewStyle 的所有类型，默认为 large white

```swift
public var chrysanStyle = UIActivityIndicatorViewStyle.whiteLarge
```

颜色，影响 icon（不包含菊花）、说明文字、进度条和进度数值的颜色

```swift
/// icon 及文字颜色，默认为白色
public var color = UIColor.white
```

支持自定义图片，图片会被强制转换成 Template 渲染模式，因此必须使用包含 alpha 通道的图片

```swift
/// 自定义的 icon 图片
public var customIcon: UIImage? = nil
```

弹出 HUD 时，可以设置遮罩以遮住背景的内容，遮罩的颜色可以自定义，默认为全透明

```swift
/// 遮罩颜色，遮挡 UI 的视图层的颜色，默认透明
public var maskColor = UIColor.clear
```

可以自定义 HUD 在视图中央的偏移，可以在某些特殊情况下调整 HUD 的位置

```swift
/// 菊花在视图中水平方向上的偏移，默认为正中
public var offsetX: CGFloat = 0
/// 菊花在视图中竖直方向上的偏移，默认为正中
public var offsetY: CGFloat = 0
```

更多内容请查看示例以及代码注释。
