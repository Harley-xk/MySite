---
title: 在 Ubuntu 系统中安装 Swift
date: 2020-02-12 22:12
---

> 简单记录一下在 Ubuntu 系统下安装 Swift 的过程

### 下载镜像

首先根据系统版本下载对应的 Swift 镜像，可以在 Swift [官方网站的下载页](https://swift.org/download/#releases)找到当前支持的系统版本。

以 Ubuntu 18.04，swift 5.14 为例，使用 wget 下载：

```bash
wget https://swift.org/builds/swift-5.1.4-release/ubuntu1804/swift-5.1.4-RELEASE/swift-5.1.4-RELEASE-ubuntu18.04.tar.gz
```

### 解压

通过上面的步骤得到的是一个 gzip 压缩包，解压：

```bash
tar zxf swift-5.1.4-RELEASE-ubuntu18.04.tar.gz
```

然后将解压得到的文件夹移动到 `/usr/share/swift`：

```bash
mv swift-5.1.4-RELEASE-ubuntu18.04 /usr/share/swift
```

### 设置 `PATH` 路径

为了方便使用，可以将 swift 的路径添加到系统的 `PATH` 路径中：

```bash
echo "export PATH=/usr/share/swift/usr/bin:$PATH" >> ~/.bashrc
# 重载 .bashrc 文件
source  ~/.bashrc
```

> 如果你使用 zsh 作为默认的命令行工具，则需要将 `.bashrc` 替换成 `.zshrc`

### 安装完成

至此 Swift 环境就安装完成了，可以通过 `swift --version` 命令来查看当前 Swift 环境的版本号，可以看到类似如下的结果：

```swift
Swift version 5.1.4 (swift-5.1.4-RELEASE)
Target: x86_64-unknown-linux-gnu
```

输入 `swift` 可以进入 swift 的执行环境：

![Swift REPL](/images/install-swift/swift-repl.png)

可以使用 `:exit` 命令退出。
