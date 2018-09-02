---
title: macOS 系统编译 Android 8.1.0 源码全过程
tags: [AOSP,Android]
---

这篇文章详细记录我在 macOS 10.13 上使用 AOSP 代码编译最新 Android 8.1.0 的 userdebug 系统，并安装到 Pixel XL 手机上，最后用 AOSP 代码成功调试 userdebug 系统 (从而可以准确将电脑上的源码和设备系统代码行号对应) 的全过程。全文包括 AOSP 的获取、编译 Android 系统、使用 Intellij IDEA 打开部分源码、使用 Intellij IDEA 进行调试这样几部分。

环境：macOS 10.13, 250GB SSD, 16GB RAM, XCode 9.2, JDK 1.8.0_151

###一、AOSP 代码的获取

因 Android 的编译需要 case-sensitive 的磁盘系统，而 macOS 的磁盘系统不满足此条件，所以我们需要建立一个 case-sensitive 的镜像文件，然后挂载此镜像文件，最后再挂载的目录中获取源码和编译源码。建立镜像文件的操作：

```bash
# 创建空间上限为 150GB 的稀疏镜像文件，创建后镜像文件其实是 ~/android.dmg.sparseimage 
# 创建后并未真正占据这么多磁盘空间，而是随着镜像中防止的数据越来越多，该镜像文件才原来越大 
hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 150g ~/android.dmg 

# 然后就可以将镜像文件挂载到 /Volumes 中，比如我挂载到 /Volumes/android 
hdiutil attach ~/android.dmg.sparseimage -mountpoint /Volumes/android 

# 解除挂载则是下面这样 
hdiutil detach /Volumes/android 

# 镜像文件的空间上限是可以更改的，比如下面更改为 160GB 
hdiutil resize -size 160G ~/android.dmg.sparseimage
```

提示一下，hdiutil 命令的某些操作需要电脑插着电源，否则会报错，不过这个报错在 Google 搜索一下就能找到答案了。

[AOSP 项目](https://source.android.com/)因为包含太多了子项目，所以每个子项目使用 Git 来管理，而整个 AOSP 则使用 repo 工具来管理，repo 其实就是一个 shell 脚本，内部调用 Git。官文提供的[获取 AOSP 代码](https://source.android.com/setup/downloading)的方式因为某些原因而比较难做到，所以我[从 USTC Mirror 来获取源码](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp)。步骤如下：

```bash
# 首先获取一下 repo 工具 
mkdir ~/bin export PATH=~/bin:$PATH 
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo 

# 先挂载，然后进入挂载后的目录，创建一个目录，我这是创建了 android-8.1.0_r20 
cd /Volumes/android 
mkdir android-8.1.0_r20 && cd android-8.1.0_r20 

# 然后使用 repo init 来初始化 AOSP 目录，因为 android-8.1.0_r20 是可以安装在 Pixel XL 上面的最新版本，所以我选择它来作为分支 
# 参见 https://source.android.google.cn/setup/build-numbers?hl=zh-cn#source-code-tags-and-builds 来查询哪些分支支持哪些设备。 
# USTC Mirror 作为同步代码的镜像 
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-8.1.0_r20 

# 接下来是需要漫长时间的一步，即从服务器同步所有代码及文件到本地。 
repo sync
```

如果 repo init 失败，可以参考一下 [USTC Mirror 的方法](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp#%E5%88%9D%E5%A7%8B%E5%90%8C%E6%AD%A5%E6%96%B9%E6%B3%952) 中的初始化仓库那一步。同步过程是可能意外中断的，此时可以直接继续执行 repo sync 来继续同步。不过，这个补刀是可以自动化执行的，那就是下面的 shell 脚本：

```bash
#!/bin/bash 
repo sync 
while [ $? -e 1 ]; do 
    repo sync 
done
echo success
```

除了同步的方式，其实还有另一条路可以走，那就是先从 USTC Mirror AOSP Monthly 或者 TUNA Mirror AOSP Monthly 下载 aosp-latest.tar 文件，然后解压到放置源码的目录，然后再 repo init，然后再 repo sync，这样的话，repo sync 就会快很多，可能一个小时就可以了吧。

### 二、编译 Android 系统

成功得到源码后，就可以编译 Android 系统镜像了。在 macOS 上，如果你不是 macOS 或 iOS 开发者的话，八成我们需要先安装 XCode。

Gentle remind：加入你也是 250GB SSD 用户，考虑到编译过程中，源码目录占据的空间会越来越庞大，那么我建议暂时将源码根目录中的 .repo 隐藏目录剪切到移动硬盘。但是光这样还不够，你还需要先解除挂载，然后再执行下面的命令来收缩稀疏镜像文件所占据的空间大小：

```bash
# 因为稀疏镜像文件删除内容时，并不会减小占据的空间大小，所以稀疏镜像文件会越来越大，解决办法就是下面这样收缩一下 
# 收缩是把稀疏镜像中没有使用的空间去除掉，从而让它不那么稀疏，就好像把海绵里的水挤掉一样 
hdiutil compact android.dmg.sparseimage
```

下面开始编译的一些步骤：

```bash
# 先进入 AOSP 根目录，然后尝试执行下面的命令，也就是清理上次构建的产物 
make clobber
```

如果上面的命令报错，提示 MacOSX SDK 缺少，那么可以 https://github.com/phracker/MacOSX-SDKs/releases 下载对应版本的压缩包，解压到 /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs 这个路径中。如果提示什么 rt.jar 找不到，那么久将 ANDROID_JAVA_HOME 设置为 JDK 的根目录。

直到 make clobber 不报错了，开始下面的步骤：
 
```bash
# 引入一个环境设置的脚本 
source build/envsetup.sh 

# 选择编译目标，我选择的是 aosp_marlin-userdebug，marlin 表示 Pixel XL，userdebug 的含义表示调试版本的 Android 系统 
lunch 

# 然后就是开始编译了，-j 后面表示多少个线程并行编译。根据电脑 CPU 硬件来选，一般 4 核心 8 线程的 CPU 就选 -j8 
make -j8
```

完整编译一遍是需要耗费若干小时时间的，所以耐心等待，我的电脑是 MacBook Pro 15 寸 2016版 2.6GHz CPU，编译大约需要三四个小时。

如果上面编译时出现类似下面的错误：

>ninja: build stopped: subcommand failed.
>
>ninja failed with: exit status 1

那么参考 https://groups.google.com/forum/#!topic/android-building/D1-c5lZ9Oco 给出的方法，需要更改 external/bison 这个项目的代码了。此时如果前面我们把 .repo 隐藏目录剪贴走了，那么现在就需要把它的一部分拷贝回来，即 external/bison 这个项目所需的 Git track 文件，通过查看 external/bison/.git 中的符号链接，我们可以知道哪些文件需要拷贝回来。

![](https://tao93.top/images/2018/09/01/1535786626.png)

红框中就是需要拷贝回来的目录。然后我们执行下面的更改：

```bash
# 先进入这个 bison 项目的目录 
cd external/bison 

# 作出更改 
git cherry-pick c0c852bd6fe462b148475476d9124fd740eba160 

# 构建一下 bison 项目 
mm 

# 回到 AOSP 根目录，然后将刚刚构建得到的 bison 替代另一个地方的 
cd ../.. && cp out/host/darwin-x86/bin/bison prebuilts/misc/darwin-x86/bison/ 

# 然后继续再编译 
make -j8
```

建议：如果你也是 250 GB SSD 用户，那么每次编译失败后，可以先 make clobber 清除构建产物，再解除挂载，然后 compat 一下稀疏镜像文件，这样可以尽可能的避免最后编译时空间不够用。

### 三、烧录到 Pixel XL 设备中

先进入开发者模式，然后打开 OEM 锁。连接到电脑，使用 adb reboot bootloader 命令让手机进入到 BootLoader 模式，当然也可以采用按设备的物理按键的方式进入 BootLoader 模式。如果此时屏幕下像下面这样，就不需要解锁，反之需要使用 fastboot flashing unlock 来解开 BootLoader 锁。糟了，今天出门没带手机，无法截图了，反正就是屏幕下部出现 unlocked 字样，然后还有一把解开的小锁的图标，就是已解锁，否则是未解锁。只有解锁后才能刷机。对了，温馨提示一下，解锁后每次开机都会提示「Your device software can't be checked for corruption. Please lock the bootloader. PRESS POWER TO PAUSE BOOT」，不用理它就好了。

现在我们要去 [Driver Binaries](https://developers.google.com/android/drivers) 下载适用于 Pixel XL 的驱动文件。这个页面很长，要在其中找到我们需要的驱动文件，需要一个细分版本号，在 [代号、标签和版本号](https://source.android.com/setup/start/build-numbers)这个页面，可以找到我之前使用的 android\-8.1.0\_r20 分支对应的细分分支是 OPM2.171019.029，使用这个细分分支，就可以在 [Driver Binaries](https://developers.google.com/android/drivers) 下载到用于 Pixel XL 的准确的驱动文件了，如下图所示，一共是两个文件，下载之后解压，发现是两个 shell 脚本，名字分别是 extract-qcom-marlin.sh 和 extract-google_devices-marlin.sh。

![](https://tao93.top/images/2018/09/01/1535786865.png)

现在，将上一步得到的两个 shell 脚本都复制到 AOSP 源码根目录，然后分别执行以下。接下来，需要以下步骤来编译得到一些额外的文件，其实我也不知道是什么文件。接下来的步骤：

```bash
source build/envsetup.sh 

# 下面这一步我依旧是选 marlin-userdebug 
lunch
make -j8
```

别担心上面这个 make 很快就会完成，因为之前已经完整编译过了。接下来要开始刷机的最后步骤了：

```bash
# 因为后面的 flashall 需要下面这样一个环境变量，所以先设置一下 
export ANDROID_PRODUCT_OUT=<AOSP_ROOT_DIR>/out/target/product/marlin 

# 进入上面这个 marlin 目录，手机保持 BootLoader 模式并连接到电脑，然后开始刷机
fastboot -w flashall
```

### 四、调试 Android 代码

user-debug 的系统安装到手机中后，可以发现和发行版还是有些不同的，比如没有 Google 框架，没有 Google Photos，甚至连时钟应用都和发行版的原生 Android 不一样。如果需要调试代码，比如我们调试到 com.android.internal.policy.PhoneWindow 这个类的代码，那么显然，需要使用一个 IDE 来打开我下载的 AOSP 源码，因为这份源码再是安装到手机中的源码，而不能使用 Android SDK 中的 sources/android-27 中的源码来调试。使用 Intellij IDEA (下面用 IDEA 简称) 来打开源码，将会比 Android Studio 好一些，因为前者更容易控制 JDK 的引入。

首先，对于我们的个人电脑来说，直接打开整个 AOSP 项目的源码显得有些考验人，同时也考验机。因为 AOSP 目录太庞大了，IDEA 建立索引将会需要非常之久，一旦文件有了更改，重新索引又要很久，此外，光是索引文件都会很大。所以权宜之计是，只把最常用的一部分 AOSP 代码 (也就是一部分子项目) 使用 IDEA 打开。我选择了 framework 目录下的所有子项目和 libcore、repo 这两个子项目。framework 是因为里面的代码常用到，libcore 是因为里面有 Java 的库代码，而 repo 是因为 repo 被引用了，所以也加进来了。

此外，因为 AOSP 源码放置在 case-sensitive 的镜像文件中，而如果 IDEA 打开位于镜像文件中的源码的话，将需要为 IDEA 配置为 case-sensitive，可是 IDEA 可能还需要打开其他的普通项目，即位于 macOS 的 case-insensitive 文件系统中的项目。这样两者就会冲突，频繁报错。所以我的方法是，把上一段提到的部分子项目拷贝到外面，也就是拷贝到 macOS 文件系统，然后再用 IDEA 打开。

接下来是具体的步骤了， 首先需要生成用于 IDEA 的 android.imp 和 android.ipr 文件:

```bash
# 在 AOSP 根目录执行下面的命令 
make idegen && development/tools/idegen/idegen.sh
```

然后就是拷贝子项目出来

```bash
# 这个 aosp_part 就是用来保存部分子项目的目录了 
mkdir aosp_part && cd aosp_part 

# 拷贝 framework 目录和 libcore 目录出来 
cp -R /Volumes/android/android-8.1.0_r20/framework . 
cp -R /Volumes/android/android-8.1.0_r20/libcore . 
cp -R /Volumes/android/android-8.1.0_r20/repo .
```
 
为避免不小心改动上面项目中的文件，我们需要 Git 来追踪它们，而所有子项目的 git 文件其实都在 .repo 这个隐藏目录中。所以：

```bash
# 接下来需要把上面子项目对应的 .git 目录拷贝过来 
# 先建一个 .repo 目录，并建好相关目录 
mkdir -p .repo/project-objects/platform && mkdir -p .repo/projects 

# 然后开始拷贝 .git 目录 
cp -R /Volumes/android/android-8.1.0_r20/.repo/project-objects/platform/frameworks .repo/project-objects/platform 
cp -R /Volumes/android/android-8.1.0_r20/.repo/project-objects/platform/libcore.git .repo/project-objects/platform 
cp -R /Volumes/android/android-8.1.0_r20/.repo/project-objects/platform/repo.git .repo/project-objects/platform 
cp -R /Volumes/android/android-8.1.0_r20/.repo/projects/frameworks .repo/projects/ cp -R /Volumes/android/android-8.1.0_r20/.repo/projects/libcore.git .repo/projects/ 
cp -R /Volumes/android/android-8.1.0_r20/.repo/projects/repo.git .repo/projects/ 

# 最后，把前面生成的两个文件拷贝过来 
cp /Volumes/android/android-8.1.0_r20/android.iml /Volumes/android/android-8.1.0_r20/android.ipr .
```

然后，就可以使用 IDEA 以打开项目的形式打开此 aosp_part 目录了。打开后，因为 libcore 中就已经有了 JDK 库代码，所以我们需要让 aosp_part 项目不使用电脑中的 JDK。方法是在 IDEA 打开的 aosp_part 项目，File → Project Structure → SDKs，然后如下图所示：

![](https://tao93.top/images/2018/09/01/1535787196.png)

点击加号，新建一个 JDK 并将新的 JDK 命名为 aosp_jdk_1.8，新建的 JDK 一开始会和已有的那个 JDK 1.8 一模一样，包含了库代码，所以我们需要把 aosp_jdk_1.8 的 ClassPath tab 中的所有 .jar 文件移除，这样 aosp_jdk_1.8 成了一个空的 JDK，将被用于 aosp_part 项目，而其他的 IDEA 项目可以依旧使用 JDK 1.8，互不影响。

在上面的窗口中，在左侧切换到 Modules tab，然后如下图，切换到 dependencies tab：

![](https://tao93.top/images/2018/09/01/1535787229.png)

在上面这个界面会有非常多的 .jar ，把它们全部移除，然后再添加前面那个 aosp_jdk_1.8，现在就已经用 IDEA 打开了部分 android 源码了：

![](https://tao93.top/images/2018/09/01/1535787252.png)

接下来是最后一步了，在 IDEA 中调试代码。

在 IDEA 中，Run → Edit Configuration → + → Remote，然后编辑一下 name，然后把 Settings 下面的 port 编辑为 8700，最后保存。

在 Android SDK 源码目录中的 tools 目录中的 monitor 可执行文件。如果这个程序打不开，别担心，先检查一下 JDK 版本是不是高于或等于 1.8.0_152，如果是，那么只需要回退到 1.8.0_151 即可。贴心的我提供一份 JDK 1.8.0_151 的下载地址，在这里找到 1.8.0_151 版本即可以。

打开 monitor 后，鼠标选中需要调试的进程：

![](https://tao93.top/images/2018/09/01/1535787290.png)

因为今天没带手机，所以上图中是个 vivo 的测试机。

然后在 IDEA 点击调试按钮 (图标是一个绿色的小虫子)，此时如果弹出 Debug tool window 并提示 connected to target VM，则说明可以加断点调试了。

![](https://tao93.top/images/2018/09/01/1535787316.png)

EOF