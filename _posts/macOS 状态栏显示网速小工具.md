---
title: macOS 状态栏显示网速小工具
tags: [macOS,swift]
---

早先我是用盗版的 [iStat Menus](https://bjango.com/mac/istatmenus/) 软件中的「状态栏显示网速」功能的，后来不再用盗版软件了，可是偶尔还是有查看网速的需求，所以我就查找 macOS 监测网速的方法，最后找到了 nettop 这个命令。

nettop 命令的参数我就不解释了，反正用它就可以看到当前时刻各个进程 download 和 upload 的字节数，所以定时执行此命令，然后可得时间区间内的上传和下载字节数，然后再除以时间间隔，就得到了网速。我定时执行的命令如下所示：

```bash
nettop -x -t wifi -t wired -k rx_dupe,rx_ooo,re-tx,rtt_avg,rcvsize,tx_win,tc_class,tc_mgt,cc_algo,P,C,R,W -P -l 1
```

执行上面的命令，就可列出若干进程和他们的上传下载字节数，下面是示例：

```
time                                interface         state        bytes_in       bytes_out
17:36:43.266151 apsd.100                                               7413           16246
17:36:43.266164 sentineld.109                                          6666            6954
17:36:43.266166 biometrickitd.253                                    113264           19802
17:36:43.266169 eoshostd.266                                              0               0
17:36:43.266171 SystemUIServer.380                                        0            3344
17:36:43.266172 rapportd.405                                            393             366
17:36:43.266174 Mail.419                                              81706           36183
17:36:43.266194 SogouServices.487                                       329             921
17:36:43.266198 mDNSResponder.602                                  20108134         3925987
17:36:43.266200 Teams.6104                                            33061           19975
17:36:43.266214 Microsoft Teams.6142                                  23322           22085
17:36:43.266220 QQ.33101                                             522914           82040
17:36:43.266222 WeChat.37612                                          53670           64492
17:36:43.266224 Google Chrome.37878                                 1633955           42648
17:36:43.266226 ss-local.38600                                        52499           74555
17:36:43.266228 netbiosd.40075                                      1008522          151841
```

以上就是最基本的东西了。因为我并不懂 swift 编程，所以我先写了一个 python 脚本，来定时执行上面的命令，然后解析命令的输出并计算网速，脚本如下：

```python
import os
import time
import subprocess

def get_time_stamp(t):
    h = int(t[0:2])
    m = int(t[3:5])
    s = int(t[6:8])
    miu_s = int(t[9:])
    return ((h * 60 + m) * 60 + s) * 1000000 + miu_s


last_result = {}
cur_result = {}
while True:
    print('---------')

    # sample, output speed
    res = subprocess.run('nettop -x -t wifi -t wired -k rx_dupe,rx_ooo,re-tx,rtt_avg,rcvsize,tx_win,tc_class,tc_mgt,cc_algo,P,C,R,W -P -l 1'.split(), stdout=subprocess.PIPE)
    # assign sample result to last result
    output = res.stdout.decode()
    cur_result = {}
    for line in output.split('\n'):
        if line == '' or line.startswith('time'):
            continue
        data = line.split()
        data_len = len(data)

        t, p, i, o = data[0], ''.join(data[1: data_len - 2]), int(data[data_len - 2]), int(data[data_len - 1])
        idx = p.rfind('.')
        p = p[:idx]
        t = get_time_stamp(t)
        cur_result[p] = (t, i, o)
    if len(last_result) > 0:
        speed = {}
        for last_p, last_d in last_result.items():
            for cur_p, cur_d in cur_result.items():
                if last_p == cur_p:
                    up_speed = cur_d[2] - last_d[2]
                    down_speed = cur_d[1] - last_d[1]
                    if cur_p in speed:
                        speed[cur_p] = (speed[cur_p][0], speed[cur_p][1] + up_speed, speed[cur_p][2] + down_speed)
                    else:
                        speed[cur_p] = (cur_p, up_speed, down_speed)
        speed = list(speed.values())
        speed.sort(key=lambda x: x[1] + x[2], reverse=True)
        for i in speed:
            print(i[0])
    last_result = cur_result
    time.sleep(1)

```

然后今天终于磕磕绊绊的用 XCode 写了一个比较简陋的「状态栏显示实时网速」的 macOS 小工具，并放在了 [github 上面](https://github.com/Tao93/NetTool)，运行效果如下图所示：

![](https://tao93.top/images/2018/09/15/1537000487.png)
