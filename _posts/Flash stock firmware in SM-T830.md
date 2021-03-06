---
title: 给三星 tablet SM-T830 刷机
tags: [Firmware, Android, Samsung]
---

#### 背景

前天公司发给我一部三星 tablet 作为测试机，是 Tab S4 wifi 版，具体型号是 SM-T830，虽然不是我期待的 Pixel 3 XL，但是配置也还不错，加上屏幕也比较大。但是坑就坑在，这是一部国行的机器，没有 Google Mobile Service (GMS) 不说，还充斥了不少国内生态的杂物，比如系统设置中的 device storage 界面，居然写着「powered by 360」，还有一些预装的国内应用。

这时摆在我前面的有三种方法：第一，刷机刷入一个原生 Android 系统，这当然是最完美的；第二，找一个三星官方的但是是国外版的系统刷入，这种系统会有 GMS；第三，自己把确实的 Google 服务安装上去，凑活着用。

以上第一种方法，我在网上查了查，几乎不可能，因为这个 机型比较小众，网上根本没有给它刷原生系统的方法，就连 [LineageOS](https://lineageos.org/) 也没有支持这个机型。第二种方法，是比较可行的，这款 tablet 即使小众，也是在全球销售的，国外绝大部分地区销售的都是带有 GMS 的，此外这是个 WiFi 版机型，也不用担心运营商和网络制式的问题，所以刷成功的话，虽不是原生 Android，但也有完整可靠的 GMS 全件套。第三种方法，改动最小，但是也可能有后遗症，因为这样自行补充安装 Google service 的方式有可能是不完备可靠的，指不定啥时候就出点 bug。

所以我就开始实施第二种方法：刷入三星官方固件的国外版本，最后这件事折腾了我一天多。

#### 下载 (错误的) 固件包

首先是获取固件包，在网上找了一通，找到了 xda 上的[这个帖子](https://forum.xda-developers.com/galaxy-tab-s4/how-to/sm-t830-t830xxu1arh1-t3831918)，里面贴出了存放在 Google drive 的适用于 SM-T830 的两个官方固件的链接，一个是法国版，另一个是阿拉伯联合酋长国版。帖子的作者还说，他对刷机可能出现的后果不负任何责任，提供这两个包只是让大家免收一些网站要收费才能下载固件包或者是付费才能提速下载的恶心。话说，SM-T830 的小众，从我搜遍 xda 也没找到几个和它相关的帖子即可略见一斑。

固件包大小约是 3.6 GB，起先我从浏览器直接下载 Google drive 上的固件包，网速很慢只有 100KB/S 的样子，下了挺久最后都失败了。后面我发现，先把固件添加到我自己的 Google drive 空间，然后在手机上的 Google drive 应用中下载，配合公司的网络环境，速度可以很快，差不多半个小时就下载好了。

#### 企图用虚拟机，失败

三星的内部刷机工具 odin，只能用 Windows 运行，我手里只有 Mac，所以我一开始希望用 virtual box 中的 win10 虚拟机来操作。我从硬盘里面把很久前备份的 win10 的 virtual disk image (也就是一个 .vdi 文件) 拷贝到电脑，然后以此新建一个虚拟机，成功的运行，但是鼓捣了半天，也无法让这个虚拟机连接 USB 设备，别说这台 tablet 不行，连 U 盘都不行。VirtualBox VM Extension Pack 安装了，在 win10 里面也安装了 Guest Additions，USB device filters 也配置了，就是不行，只能放弃，找同事借用 Windows 台式机。


**2019年1月1日更新**：我后面 VirtualBox 装 Windows 虚拟机是可以连接到 Android 设备的，并且可以刷机，见[给三星 tablet SM-T830 刷机[续]](https://tao93.top/2019/01/01/%E7%BB%99%E4%B8%89%E6%98%9F%20tablet%20SM-T830%20%E5%88%B7%E6%9C%BA[%E7%BB%AD]/) 的最后一部分


#### 失败的刷机

在同事的台式机上操作之前，我在网上已经看了一段讲解刷机过程的视频，感觉挺简单的，就几个步骤而已。等到开始在 Windows 上面开始弄，我把固件包解压，然后用 odin 开始，这才发现那个视频其实过时了，视频中固件包解压后只有一个文件，但是我解压后其实有 4 个文件。其实这一点 Odin 中都已经提示了，new model 需要 4 个文件：

![](http://tao93.top/images/2018/12/08/1544280785.png)

所以我的固件包是比较新的，所以有多个文件，但是仔细对比发现有点问题，odin 需要的是 BL, AP, CP, HOME_CSC 4 部分，但是我的固件包解压后 4 个文件的文件名是如下所示 (文件我已经删掉了，所以没法贴出完整文件名)：

> BL...
> AP...
> CSC...
> HOME_CSC...

相当于没有 CP 开头的，反而似乎有两个 CSC 的，这时我也没夺多管，依次点击 odin 中的 BL, AP, CP, CSC 四个安装，然后把 4 个文件都添加进去，然后就点 start 了，然后就失败了。于是我把 CP 和 CSC 需要的文件调换了一下，还是失败。

这时候我就查了一下为啥少了 CP 开头的文件，这才发现，原来 WiFi 版就是没有 CP 文件，这是和基带相关的，只有可插卡上网的版本才有。顺便也发现了我的 CSC... 和 HOME_CSC... 两个文件的区别，据说前者是彻底清空数据，后者会保留用户数据，所以这两个文件其实作用是类似的，其实我也注意到它们大小也非常相近，都是 180MB 左右。

于是我就空出 CP 那里不添加文件，其余 3 个添加上，然后 start，结果左边文本框停留在 'setupconnection..'，我一查，发现很多说卡在这一步的，纷纷在求 help。没办法我就只能继续查，有说要先运行 odin 后连接 USB 数据线的(试了不行)。查了一阵，终于查到一个看起来比较靠谱的[方法](https://forum.xda-developers.com/sprint-galaxy-s6/help/odin-stuck-setupconnection-t3574320)，依然是 xda 论坛上的：

![](http://tao93.top/images/2018/12/08/1544281510.png)

我按照上面说的一步一步走，依然失败，但是没再卡在 setupconnection 了。我总结发现，odin 应该确实有 bug，明明设备连接正常，它就是卡在这一步，这时候可以这么做：1. 重新启动设备，1. 断开 USB 连接，3. 进入 downloading mode，4. 连接 USB 然后再在 odin 中 start，就不会卡在那一步了。

进入 Download 模式的方法：先关机，然后同时按住 Volumn Up 和 Power 键，等屏幕亮后立即松手，此时会进入 Recovery mode (三星自己的 recovery 或者第三方 recovery 比如 TWRP)。如果是三星的 recovery，选择 boot to bootloader 即可，如果是 TWRP，选择进入 Download 模式即可。

回到刚刚说的，虽然没卡在 setupconnection 了，但是依然失败，提示大概是：

> <ID:0/004> abl.elf
> 
> <ID:0/004> FAIL! (Auth)
> 
> <ID:0/004>
> 
> <ID:0/004> Complete(Write) operation failed.
> 
> <OSM> All threads completed. (succeed 0 / failed 1)

以 abl.elf 为关键字又是一番在网上搜索，最后发现[这个帖子有](http://bbs.gfan.com/android-9245022-1-1.html)人遇到这个问题最后解决了：

![](http://tao93.top/images/2018/12/08/1544281993.png)

起初我对「固件版本落后于现有版本」这句话不以为然，因为我觉得这不就意味着只能升级不能降级嘛，不科学。但后面我忽然想到，这句话意思可能是刷机不能刷那些版本比设备出厂时系统版本还更旧的固件，就上这个人确实解决了问题，并且我前面下的固件包都是 2018 年 8 月份的，确实有点旧，所以我开始想怎么下最新的用于 SM-T830 的固件。

#### 下载最新的固件包

这时，就又回到开始找固件包的步骤了，网上满是假链接，或者是要注册，要交费的，找得人心累。不过，最后竟然找到一个叫 SamFirm tool 的神器，这个神器可以直接高速下载想要的最新固件，简直不能更棒。如下图所示，输入系统型号，然后选择 auto 模式来查找，就能找到最新的固件，并且下载速度还很快！[这个帖子](https://forum.xda-developers.com/galaxy-tab-s/general/tool-samfirm-samsung-firmware-t2988647)可以下载到这个神器。

![](http://tao93.top/images/2018/12/08/1544283275.png)

如上图所示，准确地填入型号，填入要下载的固件的国家或地区代号（这里我要填的就是某个外国或国外地区，这样固件包才会有 GMS）。这个国家或地区代号可以从[这里](https://www.sammobile.com/firmwares/galaxy-tab-s4/SM-T830/)找到，我在上一张图中填入的是 XEF，其实就是代表法国。填好后勾选 Auto，然后点击 Check Update，即可检查到最新固件包的信息，我在今天（2018年12月8日）检查到的 SM-T830 的法国版最新固件包如上图箭头所示，是 2018年10月24日更新的。然后就可以在 SamFirm 右边点下载了。建议勾选 Decrypt automatically，否则 下载好固件包后还需要重新解密，费时间。

国家或地区代号截图：

![](http://tao93.top/images/2018/12/08/1544283882.png)

#### 成功刷机

下载到最新固件后，解压，然后用 odin 依次填入 BL AP CSC 三个地方的文件，点 start，继续卡在 setupconnection，按照我之前的总结跳过这个 setupconnection，然后真正开始刷机，大概 10 分钟不到就刷好了。

#### 写在最后

刷个机遇到的坑还真是挺多，把这些记录下来，说不定以后公司的国行设备需要刷机，这些记录就能派上用场了。