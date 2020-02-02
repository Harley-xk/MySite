---
title: 基于 Cookiecutter 打造 iOS 项目模版
date: 2019-03-14 15:56
abstract: 每次新创建一个项目，我们总是要做这些事情：创建项目、初始化 pod、安装 pod、配置工程、写(或者引入)一些框架性的代码。当项目写的多了，会发现这里面基本上是大量的重复劳动，每次要做的事情几乎是一样的，但是又不可避免。特别是当有时候想写一些玩具性质的小项目时，创建项目所使用的劳动量相比就更加明显了...
tags:
 - iOS
 - Swift
 - Xcode
---

## 前言

> 每次新创建一个项目，我们总是要做这些事情：创建项目、初始化 pod、安装 pod、配置工程、写(或者引入)一些框架性的代码。当项目写的多了，会发现这里面基本上是大量的重复劳动，每次要做的事情几乎是一样的，但是又不可避免。特别是当有时候想写一些玩具性质的小项目时，创建项目所使用的劳动量相比就更加明显了。

其实可以看出来，以上这些工作其实都属于模版性质的，如果我们能够先编辑好一个项目模版，然后每次都基于这个模版创建新项目的话就好了。其实 Xcode 本身就提供了模版功能，多年前我也曾基于 Xcode 的模版功能做过一些简单的文件模版和项目模版。但是 Xcode 模版配置比较繁琐，并且官方的文档基本找不到（也可能是我没认真找）。

那么有没有什么别的好办法呢？这就引出这次要说的主角了：[Cookiecutter](https://cookiecutter.readthedocs.io/en/latest/)

## 简介

Cookiecutter 是一个用 Python 开发的项目模版工具，在 [Github](https://github.com/audreyr/cookiecutter) 上可以找到它的源码，这个项目目前已经有将近一万个 star 了。

Cookiecutter 并不是一个 iOS 专用的工具，它的支持相当广泛，基本上所有主流的项目方案都可以使用它来建立模版，这源于它十分简单粗暴的实现原理：先创建一个项目框架作为模版，将项目名称等一系列参数以关键字的方式在配置文件中指定好，然后新建项目时根据配置好的模版参数，替换模版中所有目录和子目录的名称，以及所有文本中匹配到的关键字。当然了，Cookiecutter 还有很多高级用法，可以去查询它的文档(可能需要借助梯子😔)。

这篇文章只是抛砖引玉，介绍一下我基于 Cookiecutter 创建一个简单的 App 模版的过程。

## 安装

首先需要安装 Cookiecutter，针对不同的平台和环境，Cookiecutter 提供了相当多的安装方式。不过 iOS 开发者基本都在 macOS 环境下工作，所以其他的就略过了。

mac 下官方推荐使用 Homebrew 来安装：

```bash
brew install Cookiecutter
```

整个安装过程都是自动的，非常简单。

## 创建模版

### 新建项目

首先我们需要创建一个模版：打开 Xcode，新建一个单页应用。

![新建单页应用](/images/cookiecutter/QQ20190314-142251%402x.png)

点击 **next**，然后将对应属性设置为 template 关键字：
  
```txt
Product Name: {{cookiecutter.product_name}}
Organization Name: {{cookiecutter.organization_name}}
Organization Identifier: {{cookiecutter.organization_identifier}}
```

然后点击 `next` 创建项目，如下图所示：

![创建项目](/images/cookiecutter/619631-e4d5989e654025cc.png)

项目创建完毕后，打开 info.plist，将 Bundle identifier 指定为:

```txt
com.{{cookiecutter.organization_identifier}}.{{cookiecutter.product_name}}
```

否则 Xcode 默认会将 `{` 和 `}` 替换为 `-`，导致后续创建项目时替换失败。

### 配置文件

接下来，在与刚创建完毕的项目根目录同级，也就是 `cookiecutter.product_name` 文件夹所在的目录，创建一个 **cookiecutter.json** 文件，内容如下：

```JSON
{
    "product_name": "Hello",
    "organization_name": "Someone Co.,Ltd.",
    "organization_identifier": "someone"
}
```

JSON 中的 key 就是我们刚才创建项目时填写的那几个用双括号包含起来的名称，并且去掉 `cookiecutter` 前缀，key 对应的值表示默认值。使用 Cookiecutter 从模版创建项目时，它会询问配置文件中的这几个 key，让我们给它指定名称，如果不指定，则使用配置文件中的默认值。

另外，需要哪些 key，以及 key 的名称是不固定的，你可以根据自己的需要自己定义任意数量的 key，只要保证在模版项目中使用相同的名称，**并且需要带上 `cookiecutter` 前缀**。

至此，一个最简单的项目模版就创建完了，你也可以根据自己的需要给这个项目添加一些额外的代码，新建项目时所有额外添加的内容都将原样保留。

### 钩子

上面创建的项目非常简单，但是我们实际开发中一般都需要使用 Cocoapods 来管理第三方依赖，并且有些库几乎是所有项目都需要用到的，比如 `Alamofire`、`SnapKit` 等等。那么能不能通过模版一并完成 pods 的配置和安装呢？

这就可以用上 Cookiecutter 的钩子功能了，钩子通俗一点讲也叫回调，可以在某些特性情况下触发一些事件，执行一些操作等。

Cookiecutter 的钩子配置非常简单，只需要在配置文件同级目录下新建一个 `hooks` 目录，然后将写好的脚本放在该目录中，Cookiecutter 会自动调用。

目前 Cookiecutter 支持两个事件，即创建项目之前和创建项目完成之后，会分别调用对应的脚本文件，目前支持 shell 脚本和 python 脚本。在安装前调用的脚本需要命名为 `pre_gen_project.sh` 或者 `pre_gen_project.py`；同样的，在安装之后调用的脚本需要命名为 `post_gen_project.sh` 或者 `post_gen_project.py`，取决于脚本使用的语言。

#### 自动安装 pods

首先需要在项目目录下，即 `{{cookiecutter.product_name}}/` 目录下创建 Podfile 文件，并配置好常用的依赖库：

```Ruby
target '{{cookiecutter.product_name}}' do

  use_frameworks!

  # Pods for {{cookiecutter.product_name}}

  # Networking
  pod 'Alamofire'
  # ImageCache
  pod 'Kingfisher'
  # AutoLayout
  pod 'SnapKit'

end
```

好了，不要执行 `pod install`，因为这只是个模版，可能实际创建项目时这些库已经更新了，所以需要在创建项目时才安装依赖。

接下来打开 `post_gen_project.sh`, 写上 pod 安装相关的命令：

```bash
#!/bin/bash
GREEN='\033[0;32m'
echo -e "${GREEN}Project successfuly generated!"
echo -e "${GREEN}Installing Pods..."
pod install
echo -e "${GREEN}Pods Install Finished"
```

其实只需要写一句 `pod install` 就够了，不过为了创建项目时命令行的输出内容能够更加友好，我添加了一些额外的状态输出的内容😏，你也可以在其中添加一些其他需要处理的事务。

这时候我们的项目框架才算基本完成。此时，我们的项目模版目录应该是类似下面这样：

```txt
ios_template/
├── {{cookiecutter.product_name}}/
│   ├── {{cookiecutter.product_name}}.xcodeproj
│   ├── Podfile
│   └── ...
├── hooks
│   ├── pre_gen_project.sh
│   └── post_gen_project.sh
└── cookiecutter.json
```

## 使用

### 分发模版

模版做完了，放到哪里呢？本地目录也是可以的，Cookiecutter 支持从任何可以访问的路径加载项目模版，不过为了方便使用和团队共享，最好是将模版推送到 git 仓库中，这样也方便后期修改以及多人维护等。

另外，如果你的模版项目非常大，也可以打包成 zip 上传，Cookiecutter 支持从 zip 包创建项目，创建时会自动解压。

### 创建项目

创建项目也非常简单，确保 Cookiecutter 已经正确安装后，只需要 cd 到你准备创建项目的目录，然后执行以下命令就好了：

```bash
cookiecutter https://github.com/Harley-xk/iOS_Template_Comet.git
```

>后面的路径可以是本地路径也可以是远程路径，可以是 git，也可以是 zip。Cookiecutter 还支持很多其他的仓库形式，详细可以查看它的文档。

命令执行后就开始创建项目了，Cookiecutter 会对你在配置文件中设置的每一个关键字进行询问，你可以输入新项目需要使用的值，或者也可以直接回车后使用默认值。

可以看到命令行的输出如下：

![创建项目](/images/cookiecutter/619631-d6add963dcab88b4.png)

Cookiecutter 会在用户目录下缓存所有安装过的项目模版，下次安装时会询问你是否需要更新模版：

> You've downloaded /Users/Harley-xk/.cookiecutters/iOS_Template_Comet before. Is it okay to delete and re-download it?

输入 `yes`，它会自动更新模版后创建项目。

另外可以看到在创建项目之后 `pod install` 指令也自动执行了，我们在 Podfile 里面指定的三个依赖库都已经成功安装了。可以进入 MyApp 目录下面，打开 `MyApp.xcworkspace` 查看创建完毕的项目：

![创建完毕的项目](/images/cookiecutter/QQ20190314-153222%402x.png)

## 结束语

其实可以发现 Cookiecutter 的原理和使用都非常简单。现在项目越做越庞大，架构也越来越复杂，虽然我们可以通过组件化的方式最大化的重用代码，减少工作量，但是依然需要写大量的胶水代码来整合各个组件，做大量的项目配置、项目初始化的工作，通过使用 Cookiecutter，可以进一步节省大量的工作。

*ps. 我在 Github 上创建了我自己的 iOS 项目模版，你如果不想自己写，也可以使用[我的模版](https://github.com/Harley-xk/iOS_Template_Comet.git)*
