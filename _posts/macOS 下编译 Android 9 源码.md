---
title: macOS 编译 Android 9 源码
tags: [AOSP,Android,macOS]
---

#### 环境：
    macOS High Sierra 10.13.6 with 16GB RAM
    XCode 9.4.1
    Oracle JDK 1.8.0_181

大约 1 个月之前，Google 推出了 Android Pie 正式版，我也用 Pixel XL 设备第一时间更新到了 Pie，结果发现和 Oreo 相比变化不大。此外，我也想到 Google 应该早就在 AOSP 创建了 Android Pie 的代码分支，正好我到刚到 Microstrategy 来上班，换了新的 MacBook，是时候再次用 MacBook 编译一次 AOSP 源码了。

首先我查看了手上的 2017 款 MacBook 15寸，发现这居然是 500GB 磁盘的，这令我十分惊喜，这意味着这台机器编译 Android 源码时磁盘空间将会绰绰有余。

#### 获取源代码

可从国内镜像先下载到一个压缩包，[USTC Mirror](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp) 和 [Tuna](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/) 都可选择，以下以 Tuna 为例。

首先从 [这里](https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly) 下载 aosp-latest.tar 文件和对应的 MD5 文件。这个 tar 文件中是近期的整个 AOSP 项目的压缩包。下载可以使用 wget 加 -c 命令，这样即使下载意外失败，也可以接上中断的地方继续下载。

```bash
# download aosp-latest.tar file, with -c option
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar 
```

下载完成后，可以通过以下命令获取 md5 值，检查和下载到的 aosp-latest.tar.md5 文件中的值是否一致：

```bash
# get it's md5 value then check the value
md5 aosp-latest.tar
```

另一个令人惊喜的事，最近的 aosp 体积居然减小了，之前几个月 aosp 体积那是一路蹭蹭蹭往上涨，涨到了 40 多 GB：

[](http://tao93.top/images/2018/09/06/1536224194.png)

接下来，按照之前我的 post [macOS 系统编译 Android 8.1.0 源码全过程](http://tao93.top/2018/09/01/macOS%20系统编译%20Android%208.1.0%20源码全过程/) 中的描述，建立 sparse image 并挂载，然后将 aosp-latest.tar 解压到 sparse image 中。我在 sparse image 中建立了一个 aosp 目录，然后把 aosp-latest.tar 解压到了 aosp 目录中，此时 aosp 目录中仅有一个 .repo 目录。接下来我们来把最新的 Android Pie 分支的代码 checkout。

另外，要先知道我们需要哪个分支，在 [这里](https://source.android.com/setup/start/build-numbers) 可以看到所有代码分支与代号。在这个网页如果显示为中文，就拉到最底部在左下角把语言换成英文，因为英文才有最新的 Android Pie 相关信息。如下图所示，根据自己想要用于什么设备来选择分支，比如如果是 Pixel 2 设备那么最新的分支就是 android-9.0.0_r6，而不能使 android-9.0.0_r7 和 android-9.0.0_r8，我当时用的是 android-9.0.0_r2

[](http://tao93.top/images/2018/09/06/1536224982.png)

```bash
# prepare repo
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
export PATH=${PATH}:${HOME}/bin

cd aosp

# init with specific branch android-9.0.0_r2, use other branch as you want
repo init repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r2
```

接下来就可以 repo sync 了，可以直接执行此命令，也可以像我之前 post 一样写一个 sync 脚本来执行。

有于我们下载了最新的 aosp-latest.tar，所以 sync 的过程还是比较快的，我当时应该 30 来分钟就 sync 完成了。

#### 编译

以往编译 Android 源码时，总是会出现各种各样的错误，但是这次 Android Pie 的源码编译异常顺利，没有任何错误，不知道是不是 Google 做了改进和优化。

首先下载 XCode 安装，然后进入 aosp 根目录，就可以编译了：

```bash
# clear build generated files
make clobber

# import a build script
source build/envsetup.sh

# select build target 
lunch

# start build
make -j8
```

非常顺利，3 个半小时就成功编译了，简直令人感动。相比起来，之前的 Android 源码编译会出现各种各样的错误，让人非常抓狂。
