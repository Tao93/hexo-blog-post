---
title: 通过 wifi 连接 adb
tags: [Android,adb]
---

Android 调试有个让人不开心的地方，那就是如果用真机调试，需要用数据线连接到 Android 设备，而对于 type C 接口的 MacBook，如果没有双端 type C 数据线，那么还得用转接器才行。相比较真机调试，其实还可以用模拟器，不过模拟器只能支持 x86 类型的 native 库，并且也还会有一些其他局限性。

其实，真机也是可以用局域网无线连接的，只需要电脑和 Android 设备在同一个局域网内。大致步骤就是下面几条命令：

```bash
# 先用数据线连接，确保 adb devices 可以看到设备

# 让 adb server 重新以 tcp 模式启动，端口指定为 5555
adb tcpip 5555

# 电脑和手机之间建立无线连接
adb connect <IP address of your Computer>

# 拔掉数据线，此时再检查一遍是否无线连接成功
adb devices
```

每次都想上面这样执行两三条命令显然太繁杂，尤其是其中还有一步要替换为电脑的当前内网 IP，所以应该写一个脚本来把上面的东西一键搞定：

```bash
#!/usr/bin/env bash

# 先断开一下
adb disconnect

adb tcpip 5555

# sleep 两秒，不然的话因为刚执行 adb tcpip 5555 那么后续的命令会找不到 devices
sleep 2

# 检查是否安装了黑域
adb shell pm list packages | grep 'me\.piebridge\.brevent'
has_brevent=$?
if [ $has_brevent -eq 0 ]; then
    # 重新启动黑阈
    adb -d shell sh /data/data/me.piebridge.brevent/brevent.sh
fi

# 获取 android 设备的 IP 地址
android_ip=`adb shell "ifconfig" | grep 'inet.*cast' | awk '{print $2}' | awk -F':' '{print $2}'`

echo 'Android device IP: ' $android_ip
adb connect $android_ip
```

上面的脚本中，会检查是否安装了[黑阈](https://play.google.com/store/apps/details?id=me.piebridge.brevent&hl=zh)，如果安装了，就会重新启动黑阈。黑阈是一款管理 Android 应用运行状态的应用，用来限制应用唤醒和常驻后台。Android 8.0 以上的系统，只要 USB 调试选项发生变更，黑阈就会停止起作用，所以此处需要重新让它运行起来。另外上面的脚本中还通过 ifconfig 命令来获取到电脑的内网 IP，用于执行 adb connect 命令。

