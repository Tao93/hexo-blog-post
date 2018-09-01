---
title: Homebrew 小结
tags: [Homebrew,macOS]
---

说实话，我无法适应暗色背景的界面，比如 jetbrains 系列 IDE 的 Darcula 主题 (暗色的背景)，再比如 Terminal 的一些暗色背景主题，所以我也无法直视 [Homebrew](https://brew.sh/) 主页的背景色，看得我眼睛难受。但是，Homebrew 应该是一个很不错的工具，毕竟它几乎取代了 MacPorts。

Homebrew 是 macOS 上的一款包管理工具，作用类似于 Ubuntu 上面的 apt-get。不同点是，Homebrew 的操作不需要 root 权限，Homebrew 不是系统级别的工具。Homebrew 维护一些称为 tap 的 git repository，每个 tap 里面是一些 formula，所谓的 formula 就是对应一个软件包（比图 wget 和 git 就是两个 formula）。最重要的 tap 是 homebrew/core，这个 tap 也是自带的，其中是各种命令行工具，它在安装了 Homebrew 的系统中一般位于 /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core，这可以通过 brew --repo homebrew/core 得知：

```
➜  Downloads brew --repo homebrew/core
/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
➜  Downloads cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
➜  homebrew-core git:(master) 
```

从上面可知 homebrew/core 这个 tap 在本地是一个 git 项目，实际上里面维护了四千多个 formula，也就是可供用户安装 4000 多个命令行工具，每个 formula 都是用一个 ruby 文件描述的。此外，还有 homebrew/science 和 homebrew/php 等官方 tap。除了官方 tap 外，还有著名的 caskroom/cask 这个用来安装 gui 软件(例如 Sublime Text)的 tap。插一句，在 caskroom 的 tap 中，不是用 formula 表示软件包，而是用 cask 来表示，所以大致可以认为 formula 表示命令行的包，而 cask 表示 gui 软件。对用户来说，Homebrwe 可以做的事就是搜索、安装、升级、查看、卸载各种软件包，此外也可以管理增删 tap。

除了 homebrew/core 外，我只添加了 caskroom/cask 这一个 tap。要添加此 tap 或移除它，只需要执行下面的命令：

```bash
# 添加 
brew tap caskroom/cask 

# 移除 
brew untap caskroom/cask
```

当长时间没有更新过 tap 对应的本地 git 项目，那么下一次更新时，因为国内的网络环境，更新所需要的时间可能要很久，所以我使用了 USTC Mirror 的 [Homebrew 代理](http://mirrors.ustc.edu.cn/help/brew.git.html)。具体方法其实就是，把 homebrew/core 和 caskroom/cask 这两个 tap 对应的本地 git 项目的 remote url 分别设置为 https://mirrors.ustc.edu.cn/homebrew-core.git 和 https://mirrors.ustc.edu.cn/homebrew-cask.git，这样更新就很快了。另外，brew 下载软件是从一个所谓的 bootle 去下载 (这个 bootle 的 URL 是 https://homebrew.bintray.com/，其实就是里面托管了非常多常见的软件的压缩包) 或者从软件的官网去下载。具体在哪里下载，则是根据 formula 的 ruby 描述文件中的描述来定的。

那有没有上面说的这个 bottle 的镜像以便可以快速下来软件呢？当然有，同样是来自于 USTC Mirror 的，做法很简单，设置一个环境变量 HOMEBREW_BOTTLE_DOMAIN，其值为 https://mirrors.ustc.edu.cn/homebrew-bottles 即可。

下面是这一通修改对应的命令：

```bash
# 切到 homebrew/core 对应的 git 项目目录 
cd $(brew --repo homebrew/core) 

# 换 url 
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git 

# same as previous 
cd $(brew --repo caskroom/cask) 

# same as previous 
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git 

# 我是使用 zsh，所以追加到 .zshrc 末尾就可以，如果是使用 bash，则不能这样 
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

下面开始是 brew 的一些常用命令小结。

**brew search \<formulaName\>:**

在已安装 tap 中查找包，搜索到的结果类似如果不是 homebrew/core 这个 tap 中的，将会以 tapName/formulaName 的形式列出。例如 brew search git 的结果大致是:

```
bagit
bash-git-prompt
...
git
...
caskroom/cask/git-it
```

**brew install \<formuaName\>:**

安装指定 formula 的最新版本，安装路径一般是 /usr/local/Cellar/<formulaName>/<versionNumber> 中，此外会在 /usr/local/bin 中建立指向可执行文件的符号链接，以便命令行直接运行。

**brew uninstall formulaName:**

 卸载指定 formula，不过不会删除安装包文件。
 
** brew upgrade formulaName:**

升级指定的 formula 到最新版。

**brew cleanup:**

在安装了更新版本软件包后，从 /usr/local/Cellar 删除旧版本的软件包。没错，升级时并不是覆盖式安装，而是在 /usr/local/Cellar/<formulaName> 中多了一个更高版本号的目录，然后符号链接指向更高版本目录中的可执行文件。所以，如果不想要这种冗余，就可以用 brew cleanup 来清除旧版本的目录。

**brew outdated:**

查看哪些 formula 可以升级。

**brew info formulaName:**

查看 formula 的信息，例如 brew info git, brew info caskroom/cask/git-it 等。

**brew update:**

更新所有 tap 中的包信息，其实也就是把每个 tap 对应的本地 git 项目更新到服务器最新提交，以便可以搜索到最新添加的包以及已有包的最新发布信息。

**brew list:**

列出已安装的所有包。


### tap 的管理：

**brew tap tapName:**

新增一个 tap。

**brew untap tapName:**

移除一个 tap。

**brew tap-info tapName:**

列出一个 tap 的详细信息。

更多操作，可以参考[官方文档](https://github.com/Homebrew/brew/blob/master/docs/Manpage.md)，内容很全面。
