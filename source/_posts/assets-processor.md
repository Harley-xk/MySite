---
title: 使用 Assets Processor 处理 Xcode 图片资源
date: 2017-05-26 08:27
abstract: 自从 Xcode ghost 事件之后，苹果直接在 8.0以上的版本中禁止了 Xcode 的第三方插件。苹果一直很霸道，这么做虽然确实保证了安全性，但是却同时打死了好多确实很方便和实用的插件，比如RTImageAssets。虽然苹果另外提供了 Code Editor Extension 作为补充，但是一来生态完善程度不足，这方面的扩展程序还太少，二来限制太多，很多功能无法作为代码编辑器的插件存在。
tags:
 - iOS
 - Swift
 - Xcode
---

> 自从 Xcode ghost 事件之后，苹果直接在 8.0以上的版本中禁止了 Xcode 的第三方插件。苹果一直很霸道，这么做虽然确实保证了安全性，但是却同时打死了好多确实很方便和实用的插件，比如说 [**RTImageAssets**](https://github.com/rickytan/RTImageAssets)。
> 虽然苹果另外提供了 Code Editor Extension 作为补充，但是一来生态完善程度不足，这方面的扩展程序还太少，二来限制太多，很多功能无法作为代码编辑器的插件存在。

## 硬性需求

图片资源管理是所有 App 都需要涉及到的东西，大多数时候我们会要求设计人员提供所有分辨率的切图。但是对于设计人员来说，iOS App 与 Android App 加起来需要好多套切图，这是一项比较枯燥而且繁重的工作。既然是固定模式的重复劳动，那么正是程序所擅长的，也因此诞生了  [**RTImageAssets**](https://github.com/rickytan/RTImageAssets) 这类优秀插件。

## 重生

在没有了  [**RTImageAssets**](https://github.com/rickytan/RTImageAssets) 之后写过好几个项目，深深感觉到缺少强力工具之后的无助和不便，于是我萌生了写一个与  [**RTImageAssets**](https://github.com/rickytan/RTImageAssets) 功能一致的工具 App 的想法。因为 ImageAssets 实际上是通过目录 + Contents.json 配置文件的模式运作的，所以从原理上来说脱离 Xcode 来对它进行管理是可行的，这样就可以避开 Xcode 对第三方插件的红线了。

于是便有了 AssetsProcessor，这里可耻的偷了个懒，因为  [**RTImageAssets**](https://github.com/rickytan/RTImageAssets) 是基于 MIT 协议开源的，所以 AssetsProcessor 核心的图片处理部分的代码都是从   [**RTImageAssets**](https://github.com/rickytan/RTImageAssets) 搬过来的，不过我用 Swift 重写了下。

## 使用

因为脱离了 Xcode，所以无法像原来那样在 Xcode 中直接使用快捷键来完成工作了。不过现在的用法同样简单，只需要打开 AssetsProcesor，然后将 Assets 文件夹从 Xcode 文件导航窗口中拖拽到 AssetsProcessor 的窗口中就可以了：

![](/images/assetsprocessor/AssetsProcessorExample.gif)

另外，发挥了一下桌面 App 的优势，现在处理完图片之后，会在窗口中显示每一个 ImageAsset 的处理结果，包含 1x、2x、3x 三种图片的状态，AssetsProcesor 也会将图片重命名为 n@2x.png 这种形式，并且是自动进行的。

关于图标的说明：

|图标|说明|
|---|---|
|![](/images/assetsprocessor/icn-none@2x.png)|表示原始尺寸的图片不存在，并且也没有自动生成|
|![](/images/assetsprocessor/icn-cross@2x.png)|表示处理出现错误|
|![](/images/assetsprocessor/icn-check-yellow.png)|`1x`、`2x`、`3x`：表示存在对应原始尺寸的图片，并且没有修改；<br>`name`：表示原始的文件名称符合预设风格，没有修改|
|![](/images/assetsprocessor/icn-check-green.png)|`1x`、`2x`、`3x`：表示生成了对应尺寸的图片；<br>`name`：表示已将文件名修改为预设的风格|

### 改进

AssetsProcesor 支持跨项目的图片资源处理，除了在 Xcode 中将 Assets 文件夹拖进来之外，还可以将任何目录拖拽到窗口中，AssetsProcesor 会自动检索目录中(包括子目录)所有的 Assets 目录并进行处理。

### 安装

AssetsProcesor 已经在 macOX App Store 上架，[**去下载**](https://itunes.apple.com/us/app/assets-processor/id1240024311?l=zh&ls=1&mt=12)
