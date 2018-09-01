---
title: Python 也能玩跳一跳小游戏
tags: [Python]
---

2017年12月微信发布的跳一跳小游戏，简单却又考验人，借着腾讯一贯的好友排名机制对用户的刺激，几乎是瞬间就火起来了。手残的我，玩这个游戏最多也只有几十分。看着这款小游戏，极其简单的操作 (长按就够了)、极其简单的背景 (几乎是纯色，没有任何背景装饰物)、极其简单的元素 (长方体、圆柱体等)，让我觉得应该可以用一个脚本来自动化玩这个游戏，说干就干。

对于 Android 设备，需要解决的问题其实只有：模拟长按事件、截图并获取图片、分析图片像素，分析得到每一步需要跳动的距离。下面一样一样来看。

模拟长按事件，我首先想到了 adb，查了一下，找到了 adb shell input swipe x y time 这样一条命令，swipe 本来是用来做滑动操作的，但是这里只需要长按，所以只提供了一对坐标，最后一个参数 time 表示长按的时间，OK。

截图并获取图片。获取图片好办，adb pull 一下就好了。截图的话，我也是想到了 adb，查了一下，找到了 adb shell /system/bin/screencap -p /sdcard/screenshot.png 这样的命令，用于截图并放到 sdcard 中的某个位置。

分析图片像素这一步，就仅仅剩下算法的问题啦，毕竟前面已经拿到了截图了。我是用 Python 的 PIL 图片处理库来做的。我们先来看一张跳一跳的截图：

![](http://tao93.top/images/2018/09/01/1535787465.png)

首先可以明确一个问题，其实我们不需要求两个落点之间的距离，而只要求上图中两条红色竖线之间的距离就好了，原因是不管往左上跳，还是往有上跳，这两种跳法左右对称，所以每一次跳动的距离，其实正比于两个底座的中心的 x 坐标的距离，也就是上图两条竖直红线的距离。

假设第一条红色竖线为跳之前位置，第二条竖线为下一个底座中心线。第一条竖线的 x 坐标，可以通过从上到下从左到右扫描图片每一个像素，直到找到了颜色和跳动的棋子颜色相同的颜色 (一种比较深的紫色)。然后就可以找到第一条红线的 x 坐标了。第二条竖线，可以在首次扫描到和背景颜色不一致的颜色时，此时应该就是下一个底座的最靠上的像素，并且由于底座要么是圆柱形要么是长方体，所以这个像素也就是左右方向上是居中的，也就是我们要找的第 2 条竖线的位置。

最后通过测试和调校，可以找到长按的时长和两条红色竖线距离的比值。

到此为止，思路都理清了，接下来就是写代码加细节优化了。代码中最外层是个循环，循环的每一步都是这样几步：截图、分析图片确定长按的时长、模拟长按、等待几秒等跳跃完成以免下一次循环过早开始截取到中间态的图片。

代码如下所示，下面是一个按照 1080P 屏幕写的脚本，还是我去年写的，也不知道是否还适用于现在的跳一跳，我现在也不想玩跳一跳了。

```bash
from PIL import Image
import os
import time
import random


def is_ball_color(r, g, b):
    # 判断是否是棋子顶部小球的颜色
    if r < 50 or r > 70:
        return False
    if g < 50 or g > 70:
        return False
    if b < 50 or b > 70:
        return False
    return True


def is_base_color(r, g, b):
    # 判断是否是棋子底座的颜色
    if r < 50 or r > 70:
        return False
    if g < 50 or g > 70:
        return False
    if b < 90 or b > 100:
        return False
    return True


def is_similar_color(c1, c2):
    # 判断是否是相近的颜色
    if abs(c1[0] - c2[0]) > 20:
        return False
    if abs(c1[1] - c2[1]) > 20:
        return False
    if abs(c1[2] - c2[2]) > 20:
        return False
    return True


while True:
    os.system('adb shell /system/bin/screencap -p /sdcard/screenshot.png')
    os.system('adb pull /sdcard/screenshot.png .')
    img = Image.open('screenshot.png')
    w = img.size[0]
    h = img.size[1]
    if w == 0 or h == 0:
        print('w, h:', w, h)
        break
    # print img.getpixel((310, 933))
    #  print img.getpixel((311, 933))
    #  首先是扫描来找棋子
    base_x = base_y = 0
    for y in range(400, h):
        # y 坐标从 400 开始，避免扫描到图片顶部的按钮
        x = 0
        for x in range(0, w):
            px = img.getpixel((x, y))
            if is_ball_color(px[0], px[1], px[2]):
                px = img.getpixel((x, y + 192))
                if is_base_color(px[0], px[1], px[2]):
                    break
        if x < w - 1:
            base_x = x
            base_y = y
            break
    base_y += 192
    if base_x == 0:
        print('base_x == 0')
        break
    print('base:', base_x, base_y)
    bg_px = img.getpixel((0, 400))

    # 然后是扫描来找下一个底座的位置
    dest_x = dest_y = 0
    for y in range(400, h):
        x = 0
        for x in range(0, w):
            if x % 50 == 0 and x > 0:
                px = img.getpixel((x - 50, y))
                if is_similar_color(bg_px[0 : 3], px[0 : 3]):
                    bg_px = px
                px = img.getpixel((x, y))
                if abs(px[0] - bg_px[0]) > 20 or abs(px[1] - bg_px[1]) > 20 or abs(px[2] - bg_px[2]) > 20:
                    if abs(x - base_x) > 100:
                        y += 10
                        left = right = x
                        while not is_similar_color(bg_px[0 : 3], img.getpixel((left, y))[0 : 3]):
                            left -= 1
                        while not is_similar_color(bg_px[0 : 3], img.getpixel((right, y))[0 : 3]):
                            right -= 1
                        break
        if x < w - 1:
            dest_x = (left + right) / 2
            dest_y = y
            break

    distance = abs(dest_x - base_x)
    print('dest:', dest_x, dest_y)
    x = (int)(random.random() * 200 + 600)
    y = (int)(random.random() * 200 + 600)
    # t 是长按的时间
    t = distance * 600 / 383
    loc = ' ' + str(x) + ' ' + str(y) + ' '
    os.system('adb shell input swipe ' + loc + loc + str(t))
    os.system('rm screenshot.png')
    time.sleep(3)

```
