---
title: 给三星 tablet SM-T830 刷机[续]
tags: [GSI, Project Treble]
---

事情的起因是，从 xda 上面[这个帖子](https://forum.xda-developers.com/galaxy-tab-s4/help/lineage-development-t3852509) 看到，有人在三星 Tab S4 平板上面刷了个类原生 Android 的系统，他表示用起来挺好，只不过 4 个扬声器只有底部两个可以发声。看到这个我心里就痒痒，因为三星的官方 rom 我并不喜欢，里面有三星的 app store，还有三星的一套账户系统，调试到系统代码的时候，行号也完全对不上(Pixel是可以对上行号的)，所以我也想给手里的 Tab S4 平板刷一个类原生系统。

然后我就在 xda 上面查了查，发现我应该需要先刷入一个 TWRP recovery，然后再使用 TWRP 刷入类原生系统。另外我也注意到，之所以能刷并没有适配 Tab S4 的 ROM，得归功于 Project Trable。谷歌提出 Project Trable 是为了让 Android 系统的开发和底层驱动的开发分离开，两者只要满足协议，就可以组合在一起，这样就可减小 Android 系统升级时厂商开发驱动的工作，从而让设备更快升级 Android 系统。Project Trable 也使得那些符合要求的设备，可以刷大量满足 Project Trable 要求的 General System Image (GSI)。Tab S4 就是符合 Project Trable 要求的，所以我就是要给它刷 GSI。

回到前面说的 TWRP 的事情，TWRP 官方并没有适配 Tab S4 的版本，不过我在[这个帖子](https://forum.xda-developers.com/galaxy-tab-s4/development/recovery-twrp-3-2-3-1-galaxy-tab-s4-t3843211)里面发现了用于 Tab S4 的非官方 TWRP，并写出了步骤：

1. 确认 OEM 已解锁；
2. 设备进入 download 模式；
3. 使用 odin 的 AP slot 来刷入 TWRP 压缩包，记得要取消勾选「Auto Reboot」;
4. 关机，然后再进入 TWRP；
5. 进入 TWRP，然后刷入 Forced encryption disabler patch，format DATA 一下；
6. 进入 TWRP，刷入 Magisk 压缩包，于是设备就 root 了。

步骤其实很简单，不过我在这里折腾了 3 天。我先按照上面的步骤来，结果发现进 TWRP 后就无法访问 internal storage，自然就刷不了 Forced encryption disabler patch。我试了非常多想法，比如重刷三星官方固件，刷更旧的官方固件(原因是这个帖子里面工具发出来时，官方固件还更旧一些，而我看到这帖子时已经更新到比较新了)，刷国行的官方固件，都不行，其中刷更低版本的固件还有限制即 BootLoader 不能降级，所以其实官方固件降不了多少级。我还看到了[这种帖子](https://www.thecustomdroid.com/install-twrp-samsung-galaxy-tab-s4-root-guide/)，里面告诉我说，需要先刷一个 DM-Verity patched boot，我也试了，发现刷完这个 DM-Verity patched boot 后直接无法进入系统，报 Verification Failed 错误，必须重置设备才行。翻遍 xda 那个 40 几页的帖子后，我发现其实这个帖子就是抄 xda 那个帖子，只不过抄的时候，xda 帖子上面的步骤确实第一步是刷 DM-Verity patched boot，不过后来 xda 帖子步骤更新了，成了现在这样的。

不是无法访问 internal storage 嘛，后来我买了个 micro SD 卡，插进设备，在 TWRP 中这个 micro SD 是可以访问的，所以我就可以继续刷入 Forced encryption disabler patch 并 format DATA，然后我就 boot loop 了，真气人。即使我再刷入 GSI，依然是 boot loop。

后来那个 xda 帖子的作者回复我说「全程没看到我刷 Magisk 的描述，但 Magisk 是必要的，否则会 boot loop」。我确实没刷 Magisk，因为我并没有像 root，我只想搞个 TWRP 然后刷 GSI 而已。于是我就刷了 Magisk，发现不在 boot loop 了，然后再[这里](https://github.com/phhusson/treble_experimentations/releases/tag/v108) 下载了 system-arm64-aonly-gapps-su.img.xz 这个 GSI 并刷入，这次终于成功了，进入了一个类似 AOSP user-debug 类型的 aosp 系统，很简陋，连 Contact，Settings，等这些应用都是 aosp 版本的，和正常的很不一样，相机应用则直接打开就 crash。顺便记录一下 TWRP 刷 GSI 的方法，很简单：

1. 进入 TWRP，wipe 一下；
2. 进入 install，切换到 install image 模式，找到 GSI，flash；
3. 重启

关键是，这个 aosp 系统有些 bug，最受不了的 bug 是，锁屏、旋转时屏幕都会变形一下，截屏得到的图片也是会变形一下。这和我最初看到的 xda 帖子里面那人说的只是扬声器没有全部发声差远了。所以我就想找更好些的 GSI。问了那个人，结果没回复我。

最后找到了 [Pixel Experience](https://forum.xda-developers.com/project-treble/trebleenabled-device-development/9-0-pixelexperience-p-t3833294) ，虽然依然有一些 bug，不过感觉还可以。

放几张截图：

![](http://tao93.top/images/2019/01/01/1546351524.png)
