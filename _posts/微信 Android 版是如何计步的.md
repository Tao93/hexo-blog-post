---
title: 微信 Android 版是如何计步的
tags: [Android]
---

很早以前，我使用一部 iPhone 5S 手机，这是第一款带有计步功能的 iPhone 机型，苹果称之为搭载运动协处理器。如果我没记错，iOS 的运动数据，既可以被第三方 app 读取，也可以被写入。读取好说，写入是因为用户可能使用手环之类的东西来记录数据，那么这些数据可能比 iOS 系统中的数据更受用户重视，所以就可以把 iOS 系统中的数据替换为手环产生的数据，所以需要允许写入。不过目前而言，Android 系统中，第三方应用似乎无法写入系统的运动数据，而只能读取。

回到正题，众所周知，微信有个微信运动功能。Android 版微信是怎么计步的呢？我最先是在微信的权限页发现微信声明了 Body Sensors 权限，如下所示：

![](http://tao93.top/images/2018/11/23/1542956043.png)

这个权限很有迷惑性，让人以为有这个就可以读 Android 系统产生的计步数据了。事实上，把这个权限关了，然后晃动手机 20 次，再进入微信运动，步数很准确的增加了 20 次左右，所以这个权限并不影响读取计步数据。

那这个权限可以干嘛呢？Body Sensors 权限被列为 dangerous permission，所属的 permission group 仅有这一个权限。不过很遗憾，我在网上查了挺久，没有查到获得这个权限后可以用来写什么代码做什么事情。倘若想要了解此权限的根底，估计需要在 AOSP 中找答案了。可惜我电脑上次升级失败后，目前我电脑中没有完整的 AOSP。

就目前我在网上了解到的信息而言，此权限似乎没什么用，很多帖子都在询问 Google Play Services 为何要使用此权限以及是否可以关闭该权限。我手中的 Pixel XL 也仅有 Google Play Services 和微信声明了此权限：

![](http://tao93.top/images/2018/11/24/1543033517.png)

根据网上的帖子，在 Android 7 时代的某个时候，Gmail 有个 bug，即关闭 Google Play Services 的 Body Sensors 权限后，Gmail 疯狂弹框显示「This app won't work properly unless you allow Google Play Services' request to access the foloowing: **Body Sensors**」，尽管 Gmail 仍能正常工作。因为此弹框太过怪异，随后谷歌修复了此问题，这说明即使是 Google 自己的应用，也没有严格审查自己是否声明了不必要的权限。

当然把 Google Play Services 的 Body Sensors 权限关闭后，打开 Google Fit 应用时，弹出和上面类似的消息，并且 Google Fit 不再显示我的步数，乍一看，这似乎说得通。但是，此时微信仍然可以正常计步。这说明但就计步这一功能看，Body Sensors 并非必要的。

事实上我的计步 Demo 应用如上述微信一样，也还可以计步。计步 Demo 中的代码非常简单，Google 自己也有个展示此功能的简单项目 [android-BatchStepSensor](https://github.com/googlesamples/android-BatchStepSensor)。值得一提的是，Google 的简单项目中 manifest 声明了两项 use-feature，但其实这也不是必要的。第三方应用无需声明任何权限，无需声明任何 use-feature，只需注册一个 android.hardware.SensorEventListener 接口，即可在此接口的 onSensorChanged 方法中源源不断的收到步数更新：

```java
@Override
public void onSensorChanged(SensorEvent event) {
    switch (event.sensor.getType()) {
        case Sensor.TYPE_STEP_DETECTOR:
            stepDetector ++;
            tvDetector.setText(stepDetector + "");
            break;
        case Sensor.TYPE_STEP_COUNTER:
            // event.values[0] is step count since last reboot of Android device
            stepCounter = (int) event.values[0];
            tvCounter.setText(stepCounter + "");
            break;
    }
}
```

这样第三方应用就可以计步了，即使第三方应用进程终止，也可以在应用再次运行时，在 onSensorChanged 方法中得知新的步数。

值得注意的是，系统返回的步数始终是上次重启设备后的总步数。那么第三方应用使用这种方式计步时，会存在一个问题，即应用进程被杀后，如果用户先运动，然后重启设备，然后才打开第三方应用，那么第三方应用会丢失从应用被杀到设备重启之间的步数，而只知道设备重启后新增了多少步数。

经过验证，我的 demo、微信还有 Google Fit 均存在此问题。这说明微信和 Google Fit (或者它依赖的 Google Play Services) 都是通过注册 SensorEventListener 接口来获知步数的。这也说明了微信声明 Body Sensors 权限是冗余的。

那么 Body Sensors 权限到底可以用来做什么？这是一个待填的坑。

对了，网上还有种方式，通过监听手机加速度感应器的事件来自己计步，至于准确性，那就受自己的算法的科学性和复杂性的限制了。方法也是注册 SensorEventListener，不过注册的 type 是 android.hardware.Sensor.TYPE\_ACCELEROMETER 而非前面的 TYPE\_STEP_COUNTER。更多细节见[此帖子](http://www.gadgetsaint.com/android/create-pedometer-step-counter-android/#.W_jYq5MzYkp)。


