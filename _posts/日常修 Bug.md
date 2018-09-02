---
title: 日常修 Bug
tags: [Android]
---

前几天有人报告巴西版本乘客端扫描银行卡的界面有 bug，即屏幕右侧有一条白线，也就是下图所示。

![](https://tao93.top/images/2018/09/01/1535790364.png)

鉴于报告人没提具体的版本号，也没提怎么出现的，也没提是什么机型出现的，我就和报告人说沟通了一番。报告人说应该是小米 6 出现这个问题，我遂借了一部小米 6，然后运行 demo 并没有复现。现在就有两种可能，第一是只有报告人那台小米 6 有问题，第二是 demo 没问题但是集成到巴西版本后就有问题了。经过艰苦交涉，终于从对方那里要到了一个安装包后，我手里手机复现了此问题，事实很快就清楚了：demo 没问题但是集成后有问题，也就是和机型压根没关系。

先直接给出最后修复的方法，修复方法是在我的扫卡库的 activity 中，调用以下一行就 OK 了：

```java
// 给 PhoneWindow 设置一个背景 
getWindow().setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
```

嗯，修复方法很简单，但是为什么这样做就 work 了呢？接下来就是调试的过程了.

首先获取一下这个界面的 layout 信息，尝试看看右侧白色细线是不是额外塞进去的视图，结果并不是。然后我再看是否是设置了什么 padding 导致了这个问题。事实上，正式这样的，可惜我可能是瞎了眼，居然没有看到下面的线索，即 DecorView 的右侧 padding 是 2：

![](https://tao93.top/images/2018/09/01/1535790424.png)

我只看到了下面的 LinearLayout 的宽度是1438，比我的手机屏幕宽度1440少了2个像素，显然就是右侧的白线了。

![](https://tao93.top/images/2018/09/01/1535790483.png)

所以我怎么办呢，我开始调试 measure 和 layout 的过程。因为 DecorView 的宽度是 1440 没错，可是它的 Child LinearLayout 宽度只有 1438，所以我先条件断点在 LinearLayout 类中的 onMeasure 方法。断点的条件我本来想写 getParent() instance DecorView，结果发现无法应用 DecorView，于是我就改成了 getParent() != null && getParent().getParent() == null，其实这是不对的，DecorView 同样有 Parent，也就是 ViewRootImpl 对象，我把这个给忘了。所以断点条件就成了下面这样臃肿的了：

```java
getParent() != null && getParent().getParent() != null && getParent().getParent().getParent() == null 
// 当然其实还有更简洁的方式，比如下面这样 
getClass().getName.endswith("DecorView")
```

然后发现不管是 measure 还是 layout 的过程，LinearLayout 宽度始终就是 1438，DecorView 就只给它留了这个大空间。然后我开始调试到 DecorView 的测量过程中，最后发现在 ViewGroup (这里仅仅是由于 DecorView 是继承自 ViewGroup 的) 的 measureChildWithMargins 方法中，找到了关键线索：

![](https://tao93.top/images/2018/09/01/1535790538.png)

也就是上面的 mPaddingRight 的值居然是 2 而不是 0，这意味着 DecorView 的右侧 padding 是2，这样就能解释通为啥 DecorView 的 child 宽度小了 2 了。这是我赶紧回去看 layout 信息，然后就发现了我本来早就该发现的线索了。

OK，接下来是要知道，谁把 DecorView 的右侧 padding 加了 2 的。我对 View 类的 mPaddingRight 属性加了下图所示的断点：

![](https://tao93.top/images/2018/09/01/1535790585.png)

果然，不一会儿，我就有了收获，得到了下面的堆栈：

![](https://tao93.top/images/2018/09/01/1535790614.png)

上面堆栈清晰了展示了右侧 padding 是怎么被设置为 2 的，概括来说，就是 PhoneWindow 在准备 DecorView 时，检查到有一个 Drawable，然后在设置这个 Drawable 时，去设置了 padding。那么为什么设置 Drawable 需要设置 padding 呢？看下图就明白了：

![](https://tao93.top/images/2018/09/01/1535790640.png)

原来，这是个自带 padding 的 NinePatchDrawable。接下来，我就查找了一下，发现这个 drawable 就是 PhoneWindow 中的 mBackgroundsource 属性解析而来。这个属性是从应用主题中指定的。现在回到前面我提前贴出的解决方法之一：

```java
getWindow().setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT))
```

上面的代码调用 PhoneWindow 的 setBackgroundDrawable 方法，这个调用除了设置 bg drawable 外，还直接让 mBackgroundsource 变为 0，所以后面就轮不到那个 NinePatchDrawable 了。

其实，上面的解决方法还是有点突兀，另一种方法是，在扫卡的库中设置一个主题，这样的话，就不会应用巴西版本乘客端项目中声明的主题了，自然也就不会把那个 NinePatchDrawable 设置进来。

再说点别的。这个扫卡的库，被集成到另一个库中，然后另一个库再集成到巴西版本中。中间这个库，我是没有涉足的。所以为么验证我有没有修复成功，我只能在最终的项目中再引用一下我的库经过改动后的版本，当然版本号也要增加一下。这样的话，就可以新的覆盖旧的，把中间那个库对扫卡库的引用给覆盖的。唉，真是折腾。