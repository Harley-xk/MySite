---
title: 使用 SwiftLint 进行 Swift 代码规范检查
date: 2017-03-24 16:59
abstract: SwiftLint 是 Realm 推出的一款 Swift 代码规范检查工具，基于 Github 公布的 Swift 代码规范进行代码检查，并且能够很好的和 Xcode 整合。配置好所有的设置之后，在 Xcode 中执行编译时，SwiftLint 会自动运行检查，不符合规范的代码会通过警告或者 error 的形式指示出来，并且拥有丰富的配置项，可以进行大量的自定义，相当方便。
tags:
 - Swift
 - Xcode
---

> 最近跟着公司大佬在做 Laravel 后端开发，要求使用 php lint 进行代码规范检查之后才能 push 代码，保证所有人写出风格统一的代码，方便后期的维护和 Review，于是开始往老本行上反思。
> 想想自己写了五六年的 iOS ，虽然自认代码还是写的很规整的，但是写 high 了之后还是会忽略很多细节上的东西，虽说无伤大雅，但是软件开发作为一门工程性质的东东，始终觉得规范化是一件很重要的事情。
>在之前的公司也曾经在 iOS 组内部推行过代码规范的实施，但那时候还只是停留在弄个 Word 文档，把各条规范列一列，然后开个小会普及下的程度上。现在接触了不少其他开发领域的东西，越来越觉得对于开发者来说提高视角去了解各个方面是多么重要的一件事情。不同领域的经验、做事的方式、思路，都可以相互借鉴与融合。 
>于是开始寻找在 iOS 下实行类似方案的可能性。说来也巧，最近在看 iOS 相关资料的时候发现了 SwiftLint 这玩意儿，遂打算来实践下。

## 简介

[SwiftLint](https://github.com/realm/SwiftLint) 是 [Realm](https://realm.io/) 推出的一款 Swift 代码规范检查工具，Realm 就不用介绍了，他们家推出的移动端跨平台数据库在业内的名气还是很大的，就算没有用过，相信大多数人也是听过的。
**SwiftLint** 基于 Github 公布的 [Swift 代码规范](https://github.com/github/swift-style-guide)进行代码检查，并且能够很好的和 Xcode 整合。配置好所有的设置之后，在 Xcode 中执行编译时，SwiftLint 会自动运行检查，不符合规范的代码会通过警告或者 error 的形式指示出来，并且拥有丰富的配置项，可以进行大量的自定义，相当方便。

## 安装

**SwiftLint** 有多种不同的安装方式，可以根据自己的喜好选择。

### 使用 Homebrew 安装

**Homebrew** 是 macOS 自带的包管理工具，使用这种方式安装也是最简单的：

```bash
brew install swiftlint
```

### 使用 CocoaPods 安装

通过 CocoaPods 安装同样很简单，只需要在 Podfile 中添加依赖：

```bash
pod 'SwiftLint'
```

之后执行 `pod install` 就可以自动安装了，这种方式会将 **SwiftLint** 安装到项目的 `Pods/` 目录下。如果你想要针对不同的项目使用不同的 **SwiftLint** 版本，这是一种很好的解决方案（**Homebrew** 会自动安装最新版本）。

需要注意的是使用这种方案会将整个 ** SwiftLint** 以及他的依赖包的完整资源文件都安装到 `Pods/` 目录中去，所以在使用版本管理工具比如 `git` 时要注意设置忽略相关目录。

### 使用安装包

**SwiftLint** 还支持使用 `pkg` 安装包进行安装，在官方的 Github 页面可以找到最新发布的[安装包](https://github.com/realm/SwiftLint/releases/tag/0.17.0)。

### 编译源代码

**SwiftLint** 完全使用 Swift 开发，并且它是基于 [MIT License](https://github.com/realm/SwiftLint/blob/master/LICENSE) 开源的，所以你可以下载它的源代码，然后通过以下命令编译安装：

```bash
git submodule update --init --recursive; make install
```

### 安装完成

等待安装完成，输入 `swiftlint help` 可以查看所有可用的命令：

```bash
➜  ~ swiftlint help
Available commands:

   autocorrect   Automatically correct warnings and errors
   help          Display general or command-specific help
   lint          Print lint warnings and errors (default command)
   rules         Display the list of rules and their identifiers
   version       Display the current version of SwiftLint
```

到此 **SwiftLint** 就安装完成了

## 配置

### Xcode

接下来需要在工程中配置相关编译选项，才能使 **SwiftLint** 在 Xcode 中运行起来。配置也很简单，只需要在 Xcode 的 `Build Phases` 中新建一个 `Run Script Phase` 配置项，在里面添加如下代码：

```bash
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```

如图所示：

![](https://github.com/realm/SwiftLint/blob/master/assets/runscript.png?raw=true)

如果是通过 CocoaPods 安装的 **SwiftLint** 需要将 `swiftlint` 替换为 CocoaPods 中的路径： `"${PODS_ROOT}/SwiftLint/swiftlint"`。
这里其实是设置了一个自动编译脚本，每次运行编译都会自动执行这个脚本，如果正确安装了 **SwiftLint**，就会执行 **SwiftLint** 中的代码规范检查，如果没有安装，脚本会抛出一个没有安装 **SwiftLint** 并提示下载的警告，方便提醒团队团队中没有安装的成员。
当然，你也可以设置为强制要求安装，这时如果没有安装则无法通过编译。只需要在脚本中 `echo "warning: ..."` 之后添加一行代码：`exit 1`，这样一来，如果没有安装 **SwiftLint**，编译时会直接抛出一个编译错误而非警告，提示需要安装 **SwiftLint**。
到此配置就完成了，是不是很简单。

### 自定义配置

现在编译一下项目看看，是不是很可怕😨：

![](https://upload-images.jianshu.io/upload_images/619631-bd69d1a1ac765844.png)

不要被 999+ 吓到了，仔细看一下具体的错误，会发现好多都是第三方库的代码规范问题，而且好多问题的级别被设置成为了 error
这样子可不行，第三方库的代码规范问题不能让我们自己的项目来背锅，接下来需要做一些配置，让 **SwiftLint** 在做代码规范检查的时候自动忽略 CocoaPods、Carthage 等包管理器引入的第三方库（当然，手动导入的第三方库也能设置忽略）

首先需要在项目的根目录下新建一个名为 `.swiftlint.yml` 的配置文件，输入如下内容：

```yaml
excluded:
  - Pods
```

`excluded` 配置项用来设置忽略代码规范检查的路径，可以指定整个文件夹，也可以指定精确路径下的文件，通过 `- xxxx` 的形式列在下面就可以了，比如如果你的项目使用 Carthage 管理第三方库的话，可以将 `Carthage` 目录添加到忽略列表：

```yaml
excluded:
  - Pods
  - Carthage
```

保存之后再来编译下，少了很多编译错误了，至少第三方库的编译错误都被干掉了，oh yeah~

![](http://upload-images.jianshu.io/upload_images/619631-ef9dd3879b1f6d5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
不过错误和警告依然很多，继续往下看，发现大量的 `trailing_whitespace` 的警告，这个是之前写代码不注意留下的，在空行中包含了空格，虽然肉眼看不出来，但是 **SwiftLint** 火眼金睛啊。
![](http://upload-images.jianshu.io/upload_images/619631-d4fca7a4c8043732.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
你可以选择更正所有这些不规范的问题，不过如果这个这个项目是遗留下来的老项目，可能存在大量类似的问题，手动更正这些问题需要相当多的精力，这时候可以选择开启忽略这类问题，只需要在 `.swiftlint.yml` 文件中加入如下代码：

```yaml
disabled_rules:
  - trailing_whitespace
```

再次编译，发现 `trailing_whitespace` 的问题已经不再提示了，你可以用同样的方法配置忽略特定的规则。

#### 其他规则

Xcode自动生成的代码经常包含大段的注视，我们经常会选择保留这些注释。不过 **SwiftLint** 有一个 `line_length` 的规则，默认是会检查注释的长度的，可以在 `.swiftlint.yml` 中设置忽略检查注释的长度：

```yaml
line_length:
  warning: 110
  ignores_function_declarations: true
  ignores_comments: true
```

这段代码设置了 `line_length ` 的检查规则：
`warning: 110 ` 表示单行字符数超过 110 时抛出警告，你也可以设置为其他的值。
`ignores_function_declarations` 表示是否忽略检查函数定义的长度
`ignores_comments` 设置是否忽略检查注释的长度
当然，你也可以在 `disabled_rules` 中设置忽略单行长度规则

[这里](https://github.com/realm/SwiftLint/tree/master/Source/SwiftLintFramework/Rules)有所有目前已经实现了的规则。你也可以实现自己的规则，然后给他们发 Pull requests。

本文的[最后](#SwiftLint)附上了我自己的 `.swiftlint.yml` 文件，你可以在 **SwiftLint** 的[官方文档](https://github.com/realm/SwiftLint#configuration)找到更多关于自定义规则的说明。

设置完所有配置之后，再次编译代码，之后就可以根据错误提示去更正不规范的代码了。

#### `.swiftlint.yml` 的嵌套

`.swiftlint.yml` 配置文件支持嵌套，因此

- 你可以给每个文件夹下的代码单独指定不同的规则设置
- 每个文件会匹配距离自己层级最近的父文件夹中的配置文件
- 嵌套的配置文件中的 `excluded ` 和 `included` 配置会被忽略

## 结语

终于写完了第一篇技术博客，深感写文章的不易，希望能保持下去。
这篇文章是我在初步研究了 **SwiftLint** 之后写的，一定有很多谬误和不足之处，各位轻喷- -

------
<a name="SwiftLint">`.swiftlint.yml`</a>

```yaml
disabled_rules: # rule identifiers to exclude from running
  - force_cast
  - trailing_whitespace
  - cyclomatic_complexity
  - unused_closure_parameter
#   - colon
#   - comma
#   - control_statement
# opt_in_rules: # some rules are only opt-in
#   - empty_count
#   - missing_docs
#   # Find all the available rules by running:
#   # swiftlint rules
# included: # paths to include during linting. `--path` is ignored if present.
#   - Docs.M/*/*.swift
excluded: # paths to ignore during linting. Takes precedence over `included`.
  - Carthage
  - Pods
  # - Source/ExcludedFolder
  # - Source/ExcludedFile.swift

# configurable rules can be customized from this configuration file
# binary rules can set their severity level
# force_cast: warning # implicitly
force_try:
  severity: warning # explicitly
# rules that have both warning and error levels, can set just the warning level
# implicitly
line_length:
  warning: 200
  ignores_function_declarations: true
  ignores_comments: true

# they can set both implicitly with an array
type_body_length:
  - 300 # warning
  - 400 # error
# or they can set both explicitly
file_length:
  warning: 500
  error: 1200
# naming rules can set warnings/errors for min_length and max_length
# additionally they can set excluded names
# type_name:
#   min_length: 4 # only warning
#   max_length: # warning and error
#     warning: 40
#     error: 50
#   excluded: iPhone # excluded via string
identifier_name:
  min_length: # only min_length
    error: 3 # only error
  excluded: # excluded via string array
    - id
#     - URL
#     - GlobalAPIKey
reporter: "xcode" # reporter type (xcode, json, csv, checkstyle, junit, html, emoji)
```
