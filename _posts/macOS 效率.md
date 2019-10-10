---
title: 5 分钟提高 macOS 使用效率
tags: [macOS]
---

### 前言

提起 macOS，大家可能会想到精致、广告少、软硬搭配极佳、Unix 等等，这些也是 macOS 逐渐在软件开发者、设计师、文字工作者等群里中成为「标配」的原因。然而总结我这几年的使用经验，我认为有许多工具和方法，可以在默认的 macOS 基础上，更进一步提高我们的效率。本文基于 macOS 10.14.6，不比较 macOS 和其它系统孰优孰劣，而是针对「所有用户」和「程序员」两个群体分别分享一些我的经验，欢迎讨论和交流。另外，本文推荐的几乎都是免费或者开源的工具，嗯，我就是这么为大家的钱包着想（~~我可不会说是因为我穷~~）。

### 对所有用户

#### 剪贴板管理
剪贴板管理工具帮我们维护文本和**图片**的复制历史，从而让我们能粘贴上上上上上上上上上上上上(~~请不要说我凑字数~~)...次复制的内容。显而易见这是对任何人都非常有用 enhancement，这也是为啥我把它放在第一个介绍。现在 [Alfred](https://www.alfredapp.com/) 等工具已集成了剪贴板管理的功能，不过，本着开源优先(~~免费~~)的原则，我要推荐的是 GitHub 上的 [Clipy](https://github.com/Clipy/Clipy)，安装并稍微设置后，按下设定好的快捷键，就能出现下图的弹框：

![](http://tao93.top/images/2019/08/03/1564845517.png)

然后只需要按上下方向键选择，就可以粘贴上上上上...次复制过的内容了。不过需要强调的是，在 Clipy 的[下载页面](https://github.com/Clipy/Clipy/releases)，务必下载 1.15 版本，因为 1.2.0 和 1.2.1 都有一个 bug 使得它无法将复制的图片纳入管理。

#### 软件卸载
众所周知，macOS 上安装软件一般是将一个 XXX.app 拖到一个 Applications 目录中，也就是一个拷贝的动作，完成这个后，我们就可以通过 Spotlight 搜索或者在 Launchpad 点击图标来打开它。事实上，XXX.app 其实是一个目录，而不是一个后缀为 app 的文件，这可以通过在 XXX.app 上右键，然后点击`Show Package contents`，如图：

![](http://tao93.top/images/2019/08/03/1564845703.png)

XXX.app 目录包含一个软件运行起来所需的所有文件，但当我们打开一个新安装的软件，它往往会在一些别的地方 (不在 Applications 也不在 XXX.app 中) 生成一些配置文件。所以仅仅在 Applications 中删除 XXX.app 并不能移除生成的配置文件，从而如果哪天再次安装这个软件，老的配置依然会产生作用。若要彻底卸载软件，可以使用轻量级的 [AppCleaner](https://freemacsoft.net/appcleaner/)，它可以帮我们搜索出需卸载软件的本体和所有配置文件，然后我们选择删除哪些：

![](http://tao93.top/images/2019/08/03/1564846208.png)

#### 解压缩
对于软件，其实我是能用系统自带的就用系统自带的，然而 macOS 上面的 `Archive Utility.app` 解压某些格式时会出错，也不能选择文本编码，所以就有必要使用第三方解压软件了，比如轻量的 [The Unarchiver](https://theunarchiver.com/) 了。

#### 文本编辑器

同样的，macOS 的 `TextEdit.app` 也是不堪重任，就好比是 Windows 上的记事本一样。如果你也不知道自己该用什么文本编辑器，那 [Sublime Text](http://www.sublimetext.com/) 是个很不错的选择。轻量、功能强大、定制行强、支持各种花式的插件，这就是 Sublime Text。我最喜欢它的一个功能是，在它的一个 tab 中写点东西，不用保存为文件就可以退出软件甚至关机，等下次打开它，一切内容都恢复了，这特别适合用来记点东西。

#### 窗口管理

窗口管理也就是把窗口放大缩写和挪动的操作。macOS 默认的窗口操作被一些人认为不如 Windows 好用，其实我认为稍微设置一下，就能让这些人的🤐啦。首先，macOS 其实不需要我们经常回到桌面，这有别于 Windows 的经常回到桌面点击图标启动软件。macOS 中，当需要切换到别的软件时，不应该用最小化 (快捷键 Cmd+M) 的操作，因为永远不知道最小化后露出来的是哪一个软件，所以应该直接点击或 Cmd+Tab 前往想要去的软件，除非你的某个软件真的有见不得人的东西，别人来到你电脑前时你可以疾速地按下 Cmd+M 组合键。更别提最小化后，键盘操作将不能把该窗口拉回来，而只能靠点 击 (鼠标或触摸板)，而通常键盘操作更快速。

扯远了，回到窗口管理，如果愿意付费，可以选用 [Moom](https://manytricks.com/moom/) 或 [BetterSnapTool](https://www.folivora.ai/bettersnaptool) 两款付费软件，当然他们都可免费试用。它们提供足够的功能让我们花式管理窗口，快捷键也好，触发角也罢。如果你不想大动干戈，其实我们最需要的只有两个操作：窗口最大化(不是开启全屏视图)和窗口大小回退。前者让我们将当前软件填满屏幕，从而聚精会神工作(~~摸鱼~~)；后者意思是让窗口从最大化变回本来的大小。这两个操作一般可以通过 macOS 自带的 Zoom 操作，也就是 menu bar 中 Window 菜单下的 Zoom。这个 Zoom 默认没有快捷键，不过我们可以在系统中设置：`System Preferences.app` > `Keyboard` > `Shortcuts` > `App Shortcuts`，然后点`+`按钮并如下添加：

![](http://tao93.top/images/2019/08/04/1564848618.png)

此后按上面设置的快捷键，就能最大窗口化和回退窗口了。

#### 快捷键

快捷键的世界纷繁复杂，有时候甚至不小心触发了个什么快捷键却不知情，然后一脸😳。言归正传，适用最广，使用频率最高的少数基本快捷键才是最值得提的：


快捷键 | 作用 |
:-: | :-:
Cmd + N | 一般是新建窗口 |
Cmd + T | 一般是新建 Tab 标签页
Cmd + ` (ESC 下方按键) | 在当前软件的不同窗口间切换
Cmd + Shift + [ | 当前窗口向左侧 Tab 切换
Cmd + Shift + ] | 当前窗口向右侧 Tab 切换
Cmd + W | 关闭当前 Tab 或当前窗口
Cmd + Q | 彻底退出软件
Ctrl + Cmd + Space | 卖萌时可用，试试你就知道了
Option + Cmd + C | 复制文件或目录的路径
Cmd + L | Safari 与 Chrome 中 focus 地址栏
Option + Cmd + Delete | 永久删除目录或文件
F11 | 扒开所有窗口看下桌面
Cmd + , | 进偏好设置，对几乎所有软件有效


macOS 有 Mission Control 和 App Exposé 两个功能，分别有默认的快捷键 Cmd + ↑ 和 Cmd + ↓，前者显示所有窗口，后者显示当前软件的所有窗口。当然，它们还可以在触摸板设置中通过设置手势来触发。

Windows 中出现一个弹框时，我们可以按 Tab 键来切换 focus 到不同按钮上，然后按 Enter 或空格来代替点击按钮。macOS 其实也可以，但默认未开启，开启方法是：`System Preferences.app` > `Keyboard` > `Shortcuts`，然后 `Full Keyboard Access` 选中 `All controls` 即可。

在一些大键盘上，右侧会有很多没有存在感的按键，比如 Home 和 End。其实它们的作用之一是将光标移动到行首/行末，所以它们搭配上 Shift 按键，就可以快速选中当前光标至行首/行末的文本，即快捷键 Home/End + Shift。而要选中整行，也可先 Home 然后 End + Shift 即可。macOS 中，Home 和 End 可分别由 Cmd + ← 和 Cmd + → 实现，如此一来，按下 Cmd 和 Shift 再按左右方向键即可。

有时候我们想强退一些无响应的软件，可以按 Option + Cmd + ESC 然后在窗口中强退某个软件。如果你不记得这个快捷键，通常在 Dock 中右键图标也会有强退的选项。提到 ESC，顺便说一下，绝大部分各式各样的小弹框，都可以通过按 ESC 来把弹框关闭掉。

#### 光标要快

如果你认为我说的是光标随鼠标或手指在触摸板上移动速度的快慢，那你就错啦！虽然我的确建议前面说的这个快慢可以...搞快点（毕竟这样可以减少有时候在触摸板上手指滑动一次还没到目的地，需要抬起手指接力继续滑动)，可我更想说的是**长按**键盘的左右方向键时，光标在字符之间移动的速度。首次使用 Mac 时，我就发现这个移动速度太慢，令人抓狂！幸而不就我就得知在 `System Preferences.app` > `Keyboard` 中把 `Key Repeat` 和 `Delay Until Repeat` 都设置到最右侧，就能长按左右方向键时，光标比较快地移动了。

什么？你还要更快？嗯，办法也是有的，按住 Option 的同时，长按或连击左右方向键，光标将以英文 word 为单位移动，所以移动速度会更快。

#### 默认打开方式

说到这个就想起 Xcode，但凡和代码有点关系的文件，macOS 都会默认让 Xcode 打开。发生这种情况时，我都是疾速地 Cmd+Q 退出 Xcode 然后在 Finder 中右键刚刚打开的文件，此时**按住 Option** 并在 `Always Open With` 菜单中选择我想要的软件，比如 `Sublime Text`，到这，Xcode 再也无法染指这个文件格式了 😏。

#### 简单管理启动项

Windows 中我们往往用各种电脑管家来限制某些软件开机自启的流氓行为，macOS 中这样的流氓少很多，但当真的有了的时候，或者我们偏要把某些软件设置自启 (比如我会把邮件客户端设置开机自启，这样电脑重启后，我不会忘记打开它而错过邮件通知，~~嗯，我就是热爱工作~~)。操作很简单，`System Preferences.app` > `Users & Groups` > `Login Items`，然后不想要的 login item 删掉，想增加的加上，OK。推荐把每天都用的工具软件加上去，比如前面说的 Clipy。

#### 截屏与录屏

macOS 系统自带的截屏功能，在 `System Preferences.app` > `Keyboard` > `Shortcuts` > `Screenshots` 中可一览无遗，如下图所示：

![](http://tao93.top/images/2019/08/04/1564911138.png)

有截取全屏和截取选中区域并放到剪贴板；也有截取全屏和截取选中区域并**作为文件保存**到桌面，并且这种方式下，截图刚完成时桌面右下角会和 iOS 一样短时间内显示一个缩略图，点击它的话可以编辑刚刚的截图。以上功能无法截图到剪贴板同时提供编辑功能，此外录屏功能也是缺失的，所以安装第三方截图软件势在必行。

说出来可能有点 low，其实我截屏和录屏一直是用 QQ 内嵌的截图功能。对，就是腾讯那个 QQ。简单来说，QQ 这个截图工具是可以单独运行的，并不依赖 QQ 的运行，我把这个工具按上一条「简单管理启动项」中的方法设置为开机自启。这样不用管 QQ 的死活就可以一直使用它了。

步骤是：

- 打开 QQ > `Cmd + ,` 进入设置 > `Function` > 点 `Capture & Record Screen` 右边的 `Preferences`，这时显示下面的设置界面：

![](http://tao93.top/images/2019/08/04/1564911840.png)

- 设置好你想要的截屏快捷键和录屏快捷键，并确保`Runs in background after you quit QQ` 勾选上了。
- 在 Applications 的 QQ.app 上右键再点 `Show Package Contents`，然后按如下路径找到 `QQ jietu plugin.app`，最后将它拖进 Login Items 即可：

![](http://tao93.top/images/2019/08/04/1564912051.png)

至此，按下之前设置的截图快捷键，会出现 Screenshot，Screen Recording 和 Recognition 三个选项，分别是截屏、录屏、识别图片中的文本，功能还算够用的。

![](http://tao93.top/images/2019/08/04/1564912278.png)

#### Finder

关于 Finder，有一下几点建议：

- 在 Finder 的设置中，将 `New Finder windows show` 改为使用频率最高的目录，比如 Download 或个人工作目录：

![](http://tao93.top/images/2019/08/04/1564912792.png)

- 在 Finder 的设置中，将搜索设置为搜索当前目录 (默认居然是搜索整个 Mac 😢)：

![](http://tao93.top/images/2019/08/04/1564913032.png)

- 在浏览目录或文件时，建议用下面的视图

![](http://tao93.top/images/2019/08/04/1564913106.png)

- 建议多使用 Tab 标签页，而不是打开新的 Finder 窗口。

### 对程序员

以下是面向程序员的内容~~，非战斗人员请速速撤退~~。

#### Terminal

网上很多文章一上来就是 [iTerm2](https://www.iterm2.com/)，不过我倒是觉得 macOS 自带的 Terminal 已经够强大了，并且字体更好看点，再说本着能用系统自带就不用第三方，我就一直默认使用 Terminal。

macOS 上 Terminal 窗口默认太小了，我们可以在其设置 > Profiles > Window > Window Size 中设置更大的行列数，从而充分利用我们的屏幕，省去拖拽窗口边缘的行为。

如果时常使用 Terminal，以下快捷键是非常 Life Saving 的：

快捷键 | 作用 |
:-: | :-:
Ctrl + U | 清除已输入的内容
Ctrl + A | 光标移动到行首
Ctrl + E | 光标移动到行末
Ctrl + K | 删除光标之后的内容

细心的读者可能会发现上面快捷键都是使用反人类的 Ctrl 而不是 Mac 上的 Cmd，其实上面的快捷键历史非常悠久 (那时可能还没有苹果公司)，是从早期 Unix 系统流传下来，所以同样适用于 Linux (可能会有细微差别)。另外，之前「光标要快」讲的按住 Option 时按左右方向键来快速移动光标，在此同样适用。

顺带说下可能大家不知道的两个非常有用的命令，file 和 type。前者可根据文件 meta data 判断文件类型，后者可告知你一个命令是以下来源中的哪一种：内建命令 (比如 type, cd 等)、指定路径下的可执行文件、符号链接。另外如果电脑中不小心有两个某命令(比如两个 python)且都加入了 PATH 环境变量，`type xxx` 会显示当前使用的 xxx 的路径，而 `which -a xxx` 则显示所有已加入 PATH 环境变量 的 xxx。

#### Oh My Zsh

[Oh My Zsh](https://ohmyz.sh/) 是久负盛名的开源项目，它几乎让 macOS 中 Zsh 的上手难度变为了 0。首先啰嗦一下，macOS 一直使用 bash 作为默认 Shell，这也是常见 Linux 发行版的做法。

我个人最喜欢 On My Zsh 的一下特性：

- 按已输入文本搜索历史命令。大家都知道 bash 中按向上方向键可以一条条显示历史命令，而 On My Zsh 则可以比如输入 `adb shell` 后按向上方向键，显示的全部是 `adb shell` 开头的历史命令，这让我们更快找到想要的历史命令。
- 更简单的目录选择。比如输入 ls 后，按 tab 键，会显示当前目录所有 item，然后可以直接按上方向键选中 item 再按回车确认。
- 如果当前目录在 Git 项目中，会显示当前分支，以及是否有未提交改动的小叉叉。
- 更丰富的高亮色和主题定制功能。

如果上面有你想要功能，请自行安装 😁

#### Homebrew 

[Homebrew](https://brew.sh/) 是又一个上万 star 的开源项目，大家可能早就知道了它，所以我就只简单说说。

Homebrew 自称是 macOS 上缺失的包管理器，的确如此，Linux 发行版有各种管理命令行软件包的包管理器，而 macOS 默认没有。

Homebrew 的 [core repo](https://github.com/Homebrew/homebrew-core) 为每个包维护一个 ruby 脚本文件，改文件记录了这个包的依赖信息，版本更迭信息，从哪里获取 binary 包或者源代码、编译方法等等信息。有了这些信息，Homebrew 就可以管理包的安装(包括下载源码编译的方式)、升级、卸载、依赖下载等，也就实现了包管理器的功能。这个 core repo 在我们本地也有一份，Homebrew 的 updating 其实本地的 repo pull 远程的变化，也就是这些 ruby 脚本的增删改给 pull 过来。软件包的名字是唯一性的，因而上述 ruby 脚本直接以软件包的名字命名，也是不会重复的。

值得一提的是，Homebrew 把 core repo 和后面讲到的 cask repo 叫做 tap，而 tap 里面管理的依赖包，叫做 formula，了解这些术语有利于看懂 Homebrew 的提示。

Homebrew 安装软件包，通常是在 `/usr/local` 目录中以该软件包名字为名建立目录，然后其中再以版本号为名建立目录，版本号目录中含有该版本的所有文件，且往往有个 bin 目录来包含软件包的可执行文件。也就是说，Homebrew 可以同事维护多个版本。而真正加入到 PATH 环境变量的其实是 `/usr/local/bin` 目录。Homebrew 每更新或安装一个软件包，`/usr/local/bin` 中都会更新或新建一个符号链接，来指向此软件包最新的版本的真正可执行文件。从而我们执行此符号链接，就可以运行最新的软件包。

Homebrew 管理的命令行软件，很多是 GNU 开源软件，这些软件的分发在国内有不少镜像，从而能加快我们的下载速度。为此，core repo 在国内也有镜像 (比如[中国科大的镜像](http://mirrors.ustc.edu.cn/help/brew.git.html))，只不过这个镜像并非和 Homebrew 官方的 repo 完全一致，而是下载链接指向国内的一些软件源镜像，也只有这样才能真正加快下载软件包的速度。

此外，Homebrew 也支持一些 GUI 软件 (如 Google Chrome 和 Sublime Text 等) 的管理。GUI 软件的管理，是由 [cask repo](https://github.com/Homebrew/homebrew-cask) 来支撑的。使用 Homebrew 安装 GUI 软件，通常也会在 `/usr/local/bin` 中新建符号链接来，使我们可以在 Shell 中启动 GUI 软件。比如 Sublime Text 安装好后，就会有 subl 这个符号链接：

![](http://tao93.top/images/2019/08/04/1564931295.png)

我们直接执行 `subl` 即可打开 Sublime Text，而 `subl ` 加上文件名，还可打开指定文件。

#### 正则表达式

正则表达式在我们搜索、替换结构复杂的文本时非常有用。过去，程序员只能用 grep 和 sed 命令来使用正则表达式查找、替换文本，而现在，许多 IDE 和文本编辑器都支持在查找和替换时使用正则表达式。

#### 最后的最后

效率，在日常生活中是指同样时间内完成工作的多少。追求效率是无可厚非的，然而处在这样一个快节奏的现代社会中，有时候我们也需要慢下来，悠然和缓的操持我们的电脑，享受任务完成后的一刻宁静~~，不然别人以为交给我们的任务太简单了~~。

