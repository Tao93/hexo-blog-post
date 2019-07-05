---
title: targetSdkVersion 杂谈
tags: [Android]
---

很久没有写新的博客了，原因是这一个多月以来都没想到什么合适的主题，今天就来杂谈一下 targetSdkVersion 相关的东西吧。

在我工作的第一年里，其实我并不知道 targetSdkVersion 有什么含义，查资料，看到的是类似于「代表应用针对某个 api level 完成适配了」这样的描述，这样的描述其实让我摸不着头脑。后来，我把一个小应用项目的 targetSdkVersion 升高到 api 27，这个过程才让我对 targetSdkVersion 理解比较到位了。而这时候，我发现 targetSdkVersion 的的确确如我先前查到的资料所言，表示针对某个 api level 完成适配了，这个信息是会传递给 android 系统的，android 系统依据这一信息来对应用做区别对待。

首先，需要明确 Android 系统在大版本更新时，总会加入一些新的 feature，而这些新 feature 可能会让老的应用无法完美兼容。典型的例子是 api 23 新增的运行时动态申请权限机制。在 api 23 以上的 android 系统中，系统期待代码先申请权限 (或者检查权限是已有的) 后才运行需要权限的代码 (例如拍照)。但是老应用的代码，则是只需要在 manifest 文件中申请拍照权限，就可以直接执行拍照的代码。所以，targetSdkVersion 的作用就是，如果 targetSdkVersion 大于等于 23，那么代码必须按新机制来 (即先申请权限，然后才能执行需要权限的代码)，否则，android 系统就认为这是个没有针对 api 23 进行适配的老应用 (比如可能是一个 2012 年就停止更新的应用）。对这种老应用，android 系统需要兼容，兼容的方式就是，这种应用一经安装，就自动获得了所有 manifest 中申明的权限，这样它就能在新系统中正常运行。

在新系统 (api 23 及以上) 中，对于上述的老应用，安装时会列出所有权限，告诉用户这个应用一经安装就有了下面的权限 (如同老系统安装所有应用时一样)。这就是系统的兼容机制。新系统中安装的这样的老应用后，我们在应用详情中可以看到它自动有了所有权限，可是新系统是可以手动关闭某个应用的权限的，如果我们关闭这个老应用的某个权限，会怎么样呢？如下图所示：

![](https://tao93.top/images/2018/09/03/1535985225.png)

系统会提示用户，关掉权限的话应用可能无法正常运转。这是对的，因为假设老应用中有直接执行拍照逻辑的代码，拍照权限现在被用户手动关闭，那么执行到这样的代码时就会崩溃 (without permission)！

问题是，许多应用明明一直在更新，但是为了尽量获取到各种权限，会故意将 targetSdkVersion 停留在 22 及以下，这样用户一经安装，这样的应用就自动有了它想要的任何权限，例如 Android 版手机 QQ 现在的 targetSdkVersion 依然是 17：

![](https://tao93.top/images/2018/09/03/1535985675.png)

对于这样的比较无赖的应用，其实可以放心把不想授予的权限关闭掉，它是不会崩溃的，因为其代码中其实已经做了权限检查，毕竟它一直在更新和维护。另外，对于这种现象，Google 也有措施，今年谷歌声明了对在 Google Play 更新的应用和上架的新应用都在今年必须将 targetSdkVersion 升至 27 (步子有点大)。虽然 Google Play 对国内还有点鞭长莫及，不过这一倡导应该还是会让许多大厂的应用更快的提高 targetSdkVersion。

另外，假如用户本来安装了 targetSdkVersion 为 22 的老应用，且老应用有所有想要的权限，此时如果更新到 targetSdkVersion 为 23 的新版本应用，那么新应用将「继承」所有的权限。反过来，新版本的应用的 targetSdkVersion 是不能比已安装的老版本应用还更低的，因为这不光兼容起来很麻烦，而且于情于理都不应该。 

动态申请权限大概算是最典型的一个无法完美兼容的 feature 了，除此之外，还有 api 26 引入的 adaptive icon 这个 feature。adaptive icon 使得应用可以根据 launcher 的偏好，显示圆形、圆角方形、正方形等各种形状的图标。许多老应用是直接自己裁剪一个圆角方形的图标，这显然无法完美满足 adaptive icon，所以是不完美兼容。targetSdkVersion 为 26 及以上的应用，应该且需要将 launcher icon 分为 foreground 和 background 两层，foreground 是图标中心的若干元素，比如[黑阈](https://www.coolapk.com/apk/me.piebridge.brevent) 的 launcher icon 的 foreground 是下面这样一张图片：

![](https://tao93.top/images/2018/09/03/1535986448.png)

这个图片绝大部分都是透明的，只有那 3 条弧线是灰白色的。这样一个 foreground 作为黑阈应用图标的中心元素。而 background 则是 #FF353535 这样一个纯色。事实上，黑阈的在新系统的图标是如下所示的 xml 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon
  xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_brevent_background" />
    <foreground android:drawable="@mipmap/ic_brevent_foreground" />
</adaptive-icon>
```

这样一个文件声明了 foreground 和 background。当时黑阈也是可能在 api 25 及以下的 android 系统运行的，这样的系统没有 adaptive icon 这个 feature，所以，当应用的 minSdkVersion 不到 26 时，应用中还需要为老系统准备图标资源。其实做法就是上述 xml 文件命名为 ic_launcher.xml 之类的名字，置于 drawable-anydpi-v26 这样的资源目录中，而其他用于老系统的图标资源 ic_launcher.png 文件置于 drawable-xdpi 等资源目录中，即可。

