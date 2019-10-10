---
title: 再次将 hexo blog 放置到 Github Pages
tags: [Hexo Blog]
---

#### 插话

电脑换了后，我把 hexo blog 整个目录拷贝到新电脑，然后新电脑安装 node，安装 hexo，用 npm 安装主题所需的依赖，然后 generate 出来的网页文件样式有问题，最后回退到 node v10 才解决。似乎 node 对历史版本的兼容性不佳，也可能是用 npm 安装的一些依赖没适配最新的 node，神坑！

#### 缘起

租了一年的阿里云位于香港的 VPS 到期了，但到期不是关键，关键是大约一个多月前，我的 VPS 彻底被屏蔽了，在国内 ping IP 都完全不通，只有境外才能正常访问。向阿里云反馈，对方建议我重新 deploy 一个 VPS。我以为重新 deploy 一下很简单，然而事实是重新 deploy 需要我重新下订单，然后用新订单来 deploy，原有订单就当废了。而阿里云只提供数据备份和恢复服务，用于将原 VPS 的数据迁移到新 VPS，并且这项服务也是收费的。这是在让人忍无可忍。还好原订单当时只剩一个多月，所以我打定注意不用阿里云了。

另外我简单试了一下 Vultr 的服务，先试了个 $2.5/Mo 的，结果发现只有 IPv6 而没有 IPv4。后面我申请退款，退款倒是很迅速，只是扣掉了我余额的一部分，工单里面询问，对方也只是说这个是协议规定，既然您已使用我们的产品、同意的协议，那么只能请您谅解。于是我对 Vultr 也无好感了。

加之听我的同学讲起了别的好使的翻墙方式，于是用 VPS 翻墙的需求也没什么了，于是我想到了用 Github Pages 来 host 我的 hexo blog。

#### 前奏

如我在 [博客系统迁移记录](https://tao93.github.io/2018/09/26/%E5%8D%9A%E5%AE%A2%E7%B3%BB%E7%BB%9F%E8%BF%81%E7%A7%BB%E8%AE%B0%E5%BD%95/) 中所说，我使用自己写的一个 deploy.sh 脚本，这个脚本会将我的 Markdown 文件的改动 push 到 VPS 上（以作备份，避免本地电脑数据丢失），然后调用 hexo 在 public 目录生成网页文件，并 commit 再 push 到 VPS，从而让 blog 网页刷新，最后我 blog 中用到的图片，我都是放置到 VPS，然后自动生成 image url 并插入到 Markdown 中的，因此为防 VPS 数据丢失，这个脚本还把 VPS 中的图片目录 commit，然后在本地把图片目录 pull 下来。这样一来，图片和原始的 Markdown 文件，就都在本地和 VPS 中都有完整备份，一方数据丢失时，另一方可以被用来恢复数据。

当时能有这种备份思想，也是因为曾目睹一个朋友的事故。这个朋友也是用 VPS 来 host hexo blog，而 Markdown 文件只在本地有一份，某天他本地数据丢失了，最后他不得不借助 VPS 上的网页文件来重写 Markdown 文件，而他又是 blog 高产用户，可想而知这是多么繁琐。

#### 过程

之前我有 3 git repo，分别是包含 Markdown 文件的 source，包含生成的网页文件的 public，还有就是 Image。前两者都是我本地 commit，然后 push 到 VPS，最后的 Image 则是 VPS 上不断增加新图片，然后 VPS 上 commit，本地再 pull。

现在改用 Github Pages 后，我只能有一个 repo (https://github.com/Tao93/Tao93.github.io)来 host blog 所需要的网页文件和那些图片，所以我就把 Image 全部放进 public 中了，这样就少了 Image 这个 git repo。另外，Github Pages 是 https 的，所以我还需要在 hexo config 中把 http 改为 https。

另外，为了备份 Markdown 文件，我另外开了 https://github.com/Tao93/hexo-blog-post 这个 repo 来保存我的 Markdown 文件。

往后，我更新 blog 的流程就是：本地 source repo 中新增或修改 Markdown 文件，文件中需要插入图片时，把图片放入 `public/Images` 中指定路径，然后以此路径得到 image url 并使用该 url 来插入图片，最后在 source repo 中 commit；然后 deploy.sh 脚本先将 source repo push 到 https://github.com/Tao93/hexo-blog-post，然后调用 hexo 在 public 中生成网页文件，最后将 public repo push 到 https://github.com/Tao93/Tao93.github.io，至此 blog 网页就更新了。

从上面所讲可知，Markdown 文件和 Image 都是 Github 和本地都有，互为备份，确保安全。另外上面的插入图片，依然是使用我在 [博客系统迁移记录](https://tao93.github.io/2018/09/26/%E5%8D%9A%E5%AE%A2%E7%B3%BB%E7%BB%9F%E8%BF%81%E7%A7%BB%E8%AE%B0%E5%BD%95/) 说的 upload_pic 脚本，这个脚本可以自动将我复制的图片文件或者是截图内容复制到 `public/images` 的按日期命名的目录并用时间戳命名，然后再 generate 一个生成的 url 并把 url 复制到我的剪贴板。因此我只需要复制或截图，然后再调用 upload_pic，就可以得到一个 url 用于在 Markdown 中插入图片。

最后，一年前买了阿里云 3 年的域名 tao93.top，我让它指向 tao93.github.io 了。

最后的最后，插入两张图片测试一下，一张是截图，另一张是复制图片文件。

![](http://tao93.top/images/2019/07/06/1562381892.png)

![](http://tao93.top/images/2019/07/06/1562381957.png)

<完>
