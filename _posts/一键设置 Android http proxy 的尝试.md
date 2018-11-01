---
title: 一键设置 Android http proxy 的尝试
tags: [Android]
---

自打来到杭州后，调试 Android app 时用 Charles 做代理的场景成了非常常见的操作，而让人烦恼的是，每次都需要进入手机的 WiFi -> 点击当前 WiFi -> 点击编辑 -> 点击 Advanced options -> proxy 选择 None 或者 Manual -> 上一步如果选了 Manual, 则需要输入 IP 地址和端口号 -> 保存。

这样一个六七步的步骤，真的很让人烦，而如果电脑的局域网 IP 地址不固定的话，就更加让人不爽了，意味着每次电脑重新联网后，手机都需要重新设置代理的 IP。就算电脑 IP 固定，当手机需要使用 Charles 代理或者关闭代理，都比较麻烦。所以很久前我就想有没有方法可以一键设置代理。

我的构想是，如果电脑 IP 地址不固定，那么一键设置需要在电脑上操作，不然无法获知电脑的 IP，当然也可以电脑运行个 socket server，然后手机连接，然后电脑把 IP 发给手机。如果电脑 IP 地址固定，最理想的则是手机上一键设置，这样就不需要 adb 连接了。

经过一番 google 和惨痛的尝试，我得知了两种并不完美的方法。

方法一：

```java
/**
     * 设置代理信息 exclList是添加不用代理的网址用的
     * */
    public static void setHttpProxySetting(Context context, String host, int port)
            throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException,
            IllegalAccessException, NoSuchFieldException {
        WifiManager wifiManager =(WifiManager)context.getSystemService(Context.WIFI_SERVICE);
        WifiConfiguration config = getCurrentWifiConfiguration(wifiManager);
        ProxyInfo mInfo = ProxyInfo.buildDirectProxy(host,port);
        if (config != null){
            Class clazz = Class.forName("android.net.wifi.WifiConfiguration");
            Class parmars = Class.forName("android.net.ProxyInfo");
            Method method = clazz.getMethod("setHttpProxy",parmars);
            method.invoke(config,mInfo);
            Object mIpConfiguration = getDeclaredFieldObject(config,"mIpConfiguration");

            setEnumField(mIpConfiguration, "STATIC", "proxySettings");
            setDeclardFildObject(config,"mIpConfiguration", mIpConfiguration);
            
            // save the settings
            wifiManager.updateNetwork(config);
            wifiManager.disconnect();
            wifiManager.reconnect();
        }

    }
    /**
     * 取消代理设置
     * */
    public static void unSetHttpProxy(Context context)
            throws ClassNotFoundException, InvocationTargetException, IllegalAccessException,
            NoSuchFieldException, NoSuchMethodException {
        WifiManager wifiManager = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        WifiConfiguration configuration = getCurrentWifiConfiguration(wifiManager);
        ProxyInfo mInfo = ProxyInfo.buildDirectProxy(null, 0);
        if (configuration != null){
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                configuration.setHttpProxy(mInfo);
            } else {
                Class clazz = Class.forName("android.net.wifi.WifiConfiguration");
                Class parmars = Class.forName("android.net.ProxyInfo");
                Method method = clazz.getMethod("setHttpProxy",parmars);
                method.invoke(configuration,mInfo);
                Object mIpConfiguration = getDeclaredFieldObject(configuration,"mIpConfiguration");
                setEnumField(mIpConfiguration, "NONE", "proxySettings");
                setDeclardFildObject(configuration,"mIpConfiguration",mIpConfiguration);
            }
            
            wifiManager.updateNetwork(configuration);
            wifiManager.disconnect(); 
            wifiManager.reconnect();
        }
    }
```

上面的方法直接获取到当前连接的 WiFi configuration，然后把它的 http proxy 类型 (枚举变量，主要是 NONE 和 STATIC 两个值)，和值 (即 IP 和 port) 用反射的方法设置进去，然后更新 WiFi configuration，并断开 WiFi 并重新连接 WiFi。以上设置代理和关闭代理的动作，全部由上面的代码一键完成，直接在手机上运行即可，并且代码运行结果和手机 Settings 中的 UI 结果是一致的，感觉相当完美。

然而方法一不支持 API 23 以上的 Android 系统，而所用 API 21 测试是可以完美运行的，在如今 API 28 都发布了的时候，不支持 API 23 以上可以说让实用性大打折扣。不过这是可以理解的，毕竟处于安全考虑，不能让用户的手机随便被第三方 app 默默修改了代理，这太危险了。

方法二：

```bash
adb shell settings put global http_proxy <ip>:<port>
```

执行以上命令后，Settings 中的 UI 并未更改，但是代理已经生效。这一方法实际是新增了 Settings Provider 中的 key value，有两种方式可以查看到这一新增 key value:

```bash
adb shell settings get global http_proxy
adb shell settings get global global_http_proxy_host
adb shell settings get global global_http_proxy_port
```

```java
String httpProxy = android.provider.Settings.Global.getString(contentResolver, android.provider.Settings.Global.HTTP_PROXY);
String httpProxyHost = android.provider.Settings.Global.getString(contentResolver, android.provider.Settings.Global.GLOBAL_HTTP_PROXY_HOST);
String httpProxyPort = android.provider.Settings.Global.getString(contentResolver, android.provider.Settings.Global.GLOBAL_HTTP_PROXY_PORT);
```

Android 不运行第三方应用新增这样的属性值，而只有 read 的权限，所以需要 adb 来新增这些属性值，除了这个缺点，还有更致命的缺陷。方法二的 reset proxy 的方法是：

```bash
adb shell settings delete global http_proxy
adb shell settings delete global global_http_proxy_host
adb shell settings delete global global_http_proxy_port

# 是的，目前我已知的方法，只有重启才能让上述 delete 生效，当然手动重启都可以。
adb reboot
```

要 reset proxy，删除 key value 和重启 (目前我只知道重启可以) 缺一不可，否则设备只能通过 proxy 使用 http。也就是连别的 wifi 或者使用数据流量都不能使用 http！要想不重启，除非在将 key value 设置为另一个有效的 proxy 配置，这个缺陷可以说非常致命了。


