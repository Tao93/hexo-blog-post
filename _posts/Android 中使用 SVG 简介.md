---
title: Android 中使用 SVG 简介
tags: [Android]
mathjax: true
---

#### 概述

可缩放矢量图 (Scalable Vector Graph，以下简称 SVG)，可以大大减小 Android app 中打包的图像资源大小，不用担心图片模糊的问题，很适合用于简单风格或者几何风格的图像。此外在 Android 中，SVG 极大的灵活性，远非其它 drawable 类型（[shape](https://developer.android.com/guide/topics/resources/drawable-resource#Shape)，说的就是你）能比。

事实上不光是 Android，SVG 在 Web 中也应用广泛，维基百科中大量的图表使用了 SVG，来减少网络传输资源的大小。比如维基百科的 [Android (operating system)](https://en.wikipedia.org/wiki/Android_(operating_system)) 中就有一幅 [Android Logo 图片](https://en.wikipedia.org/wiki/File:Android_new_logo_2019.svg) 是 SVG。如果我们下载这个文件，会发现它是一个文本文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 866.3 132">
   <defs>
      <style>.a{fill:#3ddc84;}</style>
   </defs>
   <title>Android1</title>
   <path d="M59.3,56.4c12.4,0,22.7,6.4,27.6,13.3V58.1h18.9v86.1H92.2a5.378,5.378,0,0,1-5.4-5.4v-6.2C82,139.6,71.7,146,59.3,146c-23.6,0-41.5-20.2-41.5-44.8S35.7,56.4,59.3,56.4m3.4,17.2C47.4,73.6,37,85.7,37,101.2s10.3,27.6,25.7,27.6c15.3,0,25.7-12.1,25.7-27.6S78.1,73.6,62.7,73.6m64.6-15.5h19V69.8c5.2-8.6,15-13.4,26.2-13.4,20,0,32.9,14.1,32.9,36v51.8H191.8a5.378,5.378,0,0,1-5.4-5.4V95.5c0-13.6-6.9-21.9-17.9-21.9-12.6,0-22.2,9.8-22.2,28.2v42.4H132.7a5.378,5.378,0,0,1-5.4-5.4Zm134.6-1.7c12.4,0,22.7,6.4,27.6,13.3V15.1h18.9V144.3H294.8a5.378,5.378,0,0,1-5.4-5.4v-6.2c-4.8,6.9-15.2,13.3-27.6,13.3-23.6,0-41.5-20.2-41.5-44.8.1-24.6,18-44.8,41.6-44.8m3.4,17.2c-15.3,0-25.7,12.1-25.7,27.6s10.3,27.6,25.7,27.6c15.3,0,25.7-12.1,25.7-27.6s-10.4-27.6-25.7-27.6m64.6-15.5h18.9V73.4a24.26,24.26,0,0,1,22.7-16.2,38.519,38.519,0,0,1,7.4.7V77.4a30.541,30.541,0,0,0-9.5-1.6c-10.9,0-20.7,9.1-20.7,26.4v42H335.1a5.378,5.378,0,0,1-5.4-5.4V58.1ZM430.1,146c-25.5,0-45.1-19.8-45.1-44.8s19.6-44.8,45.1-44.8,45.1,19.8,45.1,44.8S455.6,146,430.1,146m0-17.6c15.2,0,25.8-11.9,25.8-27.2S445.2,74,430.1,74c-15.3,0-26,11.9-26,27.2s10.7,27.2,26,27.2M500,39.3a12.783,12.783,0,0,1-12.7-12.7A12.829,12.829,0,0,1,500,14a12.65,12.65,0,0,1,0,25.3m-9.4,18.8h18.9v86.1H496a5.378,5.378,0,0,1-5.4-5.4V58.1Zm76-1.7c12.4,0,22.7,6.4,27.6,13.3V15.1h18.9V144.3H599.5a5.378,5.378,0,0,1-5.4-5.4v-6.2c-4.8,6.9-15.2,13.3-27.6,13.3-23.6,0-41.5-20.2-41.5-44.8.1-24.6,18-44.8,41.6-44.8m3.5,17.2c-15.3,0-25.7,12.1-25.7,27.6s10.3,27.6,25.7,27.6c15.3,0,25.7-12.1,25.7-27.6s-10.4-27.6-25.7-27.6" transform="translate(-17.8 -14)" />
   <path class="a" d="M822.2,111.7a9.6,9.6,0,1,1,9.6-9.6,9.6,9.6,0,0,1-9.6,9.6m-105.7,0a9.6,9.6,0,1,1,9.6-9.6,9.6,9.6,0,0,1-9.6,9.6M825.6,54.1,844.8,21a3.963,3.963,0,1,0-6.9-3.9L818.5,50.6A117.178,117.178,0,0,0,769.4,40a119.152,119.152,0,0,0-49.2,10.5L700.8,17a3.963,3.963,0,0,0-6.9,3.9L713,54a113.107,113.107,0,0,0-58.6,90.4H884.1a112.53,112.53,0,0,0-58.5-90.3" transform="translate(-17.8 -14)" />
</svg>
```

理论上讲，矢量图就是一个文本文件，里面按确定规则描述了这个图片有哪些几何元素，渲染时按照这些规则把元素一一绘制出来就好了，所以不会有模糊的问题。不过元素越复杂，越不像是几何元素，那么这个文本文件就会越大（因为需要非常多的描述），最后绘制也需要画比较多东西。

理论是很简单，不过要在 Android 中使用矢量图，我们就需要了解一下它的那些规则了。

### SVG 文本的结构

Android 中使用 SVG，我们会创建 drawable XML 文件，里面使用 `vector` 标签，如下面示例：

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="64dp"
    android:height="48dp"
    android:viewportWidth="1024"
    android:viewportHeight="768">
  <path
      android:pathData="M0,75l693,693l-693,0z"
      android:fillColor="#20FF0000" />
  <path
      android:pathData="M-215,768l1239,0l0,-1239z"
      android:fillColor="#20FF0000" />
  <path
      android:pathData="M481,768l543,-543l0,543z"
      android:fillColor="#20FF0000" />
</vector>
```
上面的 vector drawable 预览：

![](http://tao93.top/images/2020/05/16/1589666952.png)

可见 Android 用的 vector drawable 似乎和 web 用的 svg 标签格式不一样。的确不同，不过观察一下，它们都有 path 标签，正是一个一个 path 组成了矢量图的内容。而 path 中的 path data 就是一段信息量很大的文本，里面既有数字也有字母和标点。所幸这个 path data 的规则是通用的，属于 W3C 制定的一套标准，在各个平台都一样。

#### Path Data 

Path data 中的字母，被称作命令，一个命令后面跟的 0 个或多个用逗号或空格分隔的数（可以是整数也可以是小数），就是这个命令的参数。这有点像是 Unix 命令。

命令的大小写有不同。大写表示命令后面的参数当做坐标时是绝对坐标，小写时表示参数当做相对坐标（相对于该命令之前，path 已经抵达的位置点）。

最常见命令：

- M / m 命令表示移动（Move），参数是两个数，组成一对坐标。这个命令通常用来指定 path 的起点。
- L / l 命令表示画直线（Line），参数也是两个数，组成一对坐标。这个命令是最简单的指定 path 下一个位置点的方式，即显示指定下一个点的位置，从当前位置画直线过去。
- Z / z，画一条直线到起点，结束这个 path，没有参数。
- H / h，画一条水平线（Horizontal），参数是一个数，表示水平线的终点的 x 坐标。类似的命令 V / v 表示画竖直线。

实例1：

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="64dp"
    android:height="48dp"
    android:viewportWidth="1024"
    android:viewportHeight="768">
  <path
      android:pathData="M0,0 L300,300 L0,300 z"
      android:fillColor="#40FF0000" />
</vector>
```

实例2：

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="64dp"
    android:height="48dp"
    android:viewportWidth="1024"
    android:viewportHeight="768">
  <path
      android:pathData="M0,0 L300,300 l-300,0 z"
      android:fillColor="#40FF0000" />
</vector>
```

上面两个实例的效果是一样的，只不过第二个例子用到了小写的 l 命令，所以那个 -300,0 是相对坐标，即相对于 L 命令之后到达位置点的坐标。以上两个例子的预览图：

![](http://tao93.top/images/2020/05/16/1589667623.png)

#### Animated vector drawable

Android 中还支持 animated-vector 标签来创建动画矢量图。最常见的做法是，把 vector 的 pathData 当做一个属性，在动画中改变这个属性，从而让矢量图动起来。下面是一个简单的例子：

`drawable/square.xml`，定义有一个正方形的 vector drawable：

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="10dp"
    android:height="10dp"
    android:viewportHeight="400"
    android:viewportWidth="400">
    <path
        android:name="square_path"
        android:pathData="M0,0 l100,0 l0,100, l-100,0 z"
        android:fillColor="#40FF0000" />
</vector>
```

`animator/square_animator.xml`，它的 propertyName 指定了它要更改 target object 的什么属性，valueType 则指明这个 property 是一种 path：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator 
        android:duration="1000"
        android:propertyName="pathData"
        android:valueType="pathType"
        android:valueFrom="M0,0 l100,0 l0,100, l-100,0 z"
        android:valueTo="M0,0 l200,0 l0,200, l-200,0 z"/>
</set>
```

`drawable/animated_square.xml`，drawable 属性指定一个 vector drawable 为目标，而 target 标签中的 name 属性用来指定需要更改目标 vector drawable 中的哪个 path：

```xml
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/square">
    
    <target android:animation="@animator/square_animator" 
        android:name="square_path" />
</animated-vector>
```

AnimatedDrawable 类实现了 Animatable2 接口，所以调用它的 start() 方法，这个 drawable 就能动起来了：

![](http://tao93.top/images/2020/05/16/1589684164.gif)

#### 椭圆弧

有了 L 命令，我们可以画三角形、平行四边形、多边形等等。可以指定我们要画的东西占据 android drawable 的多大空间。接下来记录一下圆弧的简单绘制方式。有了这些，我们就能做许多其它 drawable 类型不能做的事了。

在 web svg 中，圆是很简单的，只需要圆心坐标和半径就可以。但是 Android vector drawable 中更复杂。需要两段圆弧拼成一个圆。不过这个复杂性正好引出了**椭圆弧**的绘制规则，也就是 A / a 命令（elliptical **a**rc）。

A / a 命令有 7 个参数，这 7 个参数依序命令为 rx, ry, x-axis-rotation, large-arc-flag, sweep-flag, x, y.

- rx 和 ry 表示椭圆的长半轴和短半轴（也就是高中数学里的椭圆方程 $$\frac{x^2}{a^2} + \frac{y^2}{b^2}  = 1$$ 中的 $$a$$ 和 $$b$$）.
- x-axis-rotation 表示椭圆的长轴相对于 x 轴的夹角，单位是弧度单位的 °，负数表示逆时针旋转长半轴，正数表示顺时针旋转。

中途解释：

rx 和 rx 确定了椭圆的大小和形状，但没确定位置。当椭圆弧的起点 （path 的当前位置点）和终点（最后两个参数）固定后，大小和形状固定的椭圆需要经过这两个点，但椭圆本身依然可以在这两个点上滑动和翻转（只要翻转后长轴方向没变），所以才需要 x-axis-rotation 参数确定一个角度，从而椭圆不能滑动。

然而，在角度固定时，下图中 1, 2, 3, 4 所指的 4 条弧表明，还是有 4 种情况，这时候就需要参数 large-arc-flag 和 sweep-flag 来四选一了。

![](http://tao93.top/images/2020/05/16/1589674930.png)

- large-arc-flag 这个参数为 0 时表示这个弧度是小角度弧度，为 1 表示大角度弧度。上图中 1 和 3 所指的两条弧为小角度，而 2 和 4 所指为大角度。
- sweep-flag 这个参数，为 0 表示逆时针画弧，1 表示顺时针画弧。上图中 1 和 2 是顺时针，而 3 和 4 则是逆时针画弧。
- x, y 这两个参数是椭圆弧终点的坐标。

有了椭圆弧，就可以拼接两段椭圆弧形成椭圆，而圆只是特殊的椭圆。下面是一个椭圆的例子。

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="10dp"
    android:height="10dp"
    android:viewportHeight="400"
    android:viewportWidth="400">
    <path
        android:name="square_path"
        android:pathData="M0,200 A100,60 0 0 1 400,200 
        M0,200 A100,60 0 0 0 400,200 z"
        android:fillColor="#40FF0000" />
</vector>
```

上面实例中，两段椭圆弧，都是从 (0, 200) 到 (400, 200)，唯一不同点是，其中一段顺时针，另一段逆时针，从而正好构成一个椭圆：

![](http://tao93.top/images/2020/05/16/1589686316.png)

### 实践

给公司 app 的桌面图标做的 vector drawable，其中的 clip path 使得只有这个区域范围内的东西才会显示出来，从而达成一个远交矩阵的效果：

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="27dp"
    android:height="27dp"
    android:viewportWidth="248"
    android:viewportHeight="248">

    <!-- clip path which is the big round corner rectangle, thus only inside this area of other parts would be shown -->
    <clip-path
        android:pathData="
            M0,42 a42,42 0 0 1 42,-42 h164
            a42,42 0 0 1 42 42 v164
            a42,42, 0 0 1 -42 42 h-164
            a42,42, 0 0 1 -42 -42 z" />
    
    <!-- the blue background of the whole icon -->
    <path android:fillColor="@color/dossier_icon_bg"
        android:pathData="M0,0 h248 v248 h-248 z" />
    
    <!-- the left rectangle of the center part. 
    the path data consists of outer clock-wise rect, and 2 inner anti-clock-wise rect. -->
    <path
        android:fillColor="@color/white"
        android:pathData="
            M62,54 l2,-2 h32 l2,2 v121 l-2,2 h-32 l-2,-2 v-121
            m7,20 v6 l1,1 h19 l1,-1 v-6, l-1,-1 h-19 l-1,1
            m0,14 v6 l1,1 h19 l1,-1 v-6, l-1,-1 h-19 l-1,1 z"/>

    <!-- upper part of the middle column of the center part -->
    <path
        android:fillColor="@color/white"
        android:pathData="M112,95 l2,-2 h24 l2,2 v12 h-28 z"/>

    <!-- middle part of the middle column of the center part -->
    <path
        android:fillColor="@color/white"
        android:pathData="M112,115 h28 v42 h-28 z"/>

    <!-- bottom part of the middle column of the center part -->
    <path
        android:fillColor="@color/white"
        android:pathData="M112,163 h28 v12 l-2,2 h-24 l-2,-2 z"/>

    <!-- the right rectangle of the center part. 
    the path data consists of outer clock-wise rect, and 2 inner anti-clock-wise rect. -->
    <path android:fillColor="@color/white"
        android:pathData="
            M154,67 l2,-2 h32 l2,2 v108 l-2,2 h-32 l-2,-2 v-108
            m7,40 v6 l1,1 h19 l1,-1 v-6, l-1,-1 h-19 l-1,1
            m0,14 v6 l1,1 h19 l1,-1 v-6, l-1,-1 h-19 l-1,1 z" />

    <!-- the bottom white line of the center part -->
    <path android:fillColor="@color/white"
        android:pathData="M48,183 l1,-1 h154 l1,1 v6 l-1,1 h-154 l-1,-1 z" />

    <!-- the red area shape -->
    <path android:fillColor="#d9242e"
        android:pathData="M0,160 h44 a44,44 0 0 1 44,44 v44 h-88 z" />

    <!-- the 3 small columns above the red area -->
    <path android:fillColor="@color/white"
        android:pathData="M20, 180 h11 v40 h-11 z" />
    <path android:fillColor="@color/white"
        android:pathData="M39, 180 h11 v40 h-11 z" />
    <path android:fillColor="@color/white"
        android:pathData="M58, 180 l11,11 v29 h-11 z" />

</vector>
```

上面的 vector drawable 的效果就是[这个 app](https://play.google.com/store/apps/details?id=com.microstrategy.android.dossier) 的图标了（当然，是带圆角的）。

#### 其它

除了使用 fill color 来填充 path 内部，还可以用 stroke color 和其它属性来控制笔画的颜色和样式。矢量图规则是非常庞大和复杂的。可以参考 Android 开发文档，或者是 [W3C 的文档](https://www.w3.org/TR/SVG/)。

矢量图在 Android 中的应用已经相当广泛了，只需扫一眼 Google 系列的各个 app，就会发现它们的图标都是扁平化的几何风格，如果解压 apk 再去看 app 桌面图标资源文件，就能确定他们的确用的是矢量图。乔布斯之后，扁平化风格大行其道，这可能也是矢量图的一阵春风吧。

#### 参考资料

[Convert VectorDrawable to SVG](https://stackoverflow.com/questions/44948396/convert-vectordrawable-to-svg)

[SVG | MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG)

[Android Vector Drawables Fundamentals](https://caster.io/courses/android-vector-drawables-fundamentals)

[Scalable Vector Graphics (SVG) 2](https://www.w3.org/TR/SVG/)
