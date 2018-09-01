---
title: Linux 执行远程脚本
tags: [Shell,Python]
---

最近工作上需要写一套跨服务器分隔数据和转移数据的脚本。经过实践，发现单纯执行在本地执行命令来操控远程服务器的数据不现实。所以需要把脚本放在远程服务器，然后本地让服务器自己去执行脚本。此外，需要使用 expect 工具来自动输入远程服务器登录密码。

先提一些  expect 的用法，即可以写成 expect 脚本形式 (即脚本以 #!/usr/bin/expect 开头)，也可以写成 expect -c '<several_cmd>' 的单条命令形式。后者的示例形式：

```bash
expect -c 'spawn scp root@example:~/xx.txt .; expect "password:"; send "abc1234\\r"; interact;'
```

上面例子中，spawn 后面表示需要执行的命令 (此处是一个 scp 命令，需要输入密码，这就是为什么要 expect 的原因)，expect 后面是表示标准输出出现 "password:" 字样时，就自动输入 send 后面的东西，也就是密码。最后一个 interact 表示接下来允许用户自己交互。

本来直接使用 shell 脚本就可以的。不过我这设计到数据的切割，所以就使用了 Python，然后需要用到 shell 命令时，是由 Python 脚本再调用 shell 命令的。

```python
import os


def call_remote_cmd(host, password, cmd):
    """
    让远程服务器自己执行命令 
    :param host: 远程服务器用户名加域名，格式示例：root@example.com 
    :param cmd: 要让远程服务器执行的命令 
    :return: 一些结果，其中可能有一些提示性文本
    """
    # 举个例子，ssh root@example.com ls 可以让远程服务器执行 ls 命令
    final_cmd = '/usr/bin/expect -c \'spawn ssh %s %s; expect "*Password:"; send "%s\\r"; interact;\'' % (host, cmd, password)
    res = os.popen(final_cmd).read()
    print(res)
    return res 


# 传给远程服务器上的 Python 脚本作为参数 
args = {'a': 1, 'b': [1, 2, 3]}

# args 需要经过复杂的替换，才能保证准确传递到远程服务器的 Python 脚本中
args = str(args) 
args = args.replace('\'', '\\\"')
args = args.replace(' ', '\\ ') 
args = args.replace(',', '\\,') 
args = args.replace('[', '\\[') 
args = args.replace(']', '\\]') 
args = args.replace('{', '\\{') 
args = args.replace('}', '\\}') 
args = args.replace(';', '\\;') 
args = args.replace('"', '\\"') 

cmd = '/usr/local/bin/python3 x.py \\"%s\\"' % args 
call_remote_cmd('root@example.com', 'password', cmd)

```

上面是我因工作需要而写出来的让远程服务器执行它的 Python 脚本的本地 Python 脚本代码。使用的时候就简单了。服务器上面放一个被调用的 Python 脚本。然后执行上面代码，就可以了。

想必直接写 shell 脚本的话，可能会更简洁些。等啥时候有空了试试看。
