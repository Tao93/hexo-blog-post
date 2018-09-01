---
title: 常用的 Bash 命令总结
tags: [Linux,Bash]
---

这篇记录是我看 [The Linux Command Line](http://linuxcommand.org/tlcl.php) 后为了总结而写的，目的是便于自己随时查阅。The Linux Command Line 是我相当喜欢的一本电子书，它是一本值得一页一页看下去的书。

**cp**

```bash
# 仅复制目标目录不存在或者存在但是更旧的的项。 
cp -u 

# 如果 file2 已经存在, file2 的内容会被 file1 的内容重写。如果 file2 不存在，则会创建它。 
cp file1 file2 

# 如果目录 dir2 不存在，创建目录 dir2，操作完成后，目录 dir2 中的内容和 dir1 中的一样。如果目录 dir2 存在，则目录 dir1 和其中的内容将会被复制到 dir2 中。 
cp -r dir1 dir2
```

**mv**

```bash
# 仅移动目标目录不存在或者存在但是更旧的的项。 
mv -u 

# 如果目录 dir2 不存在，创建目录 dir2，并且移动目录 dir1 的内容到目录 dir2 中，同时删除目录 dir1。如果目录 dir2 存在，移动目录 dir1( 它的内容)到目录 dir2 中。 
mv dir1 dir2
```

**rm**

```
# 警惕不小心写出 rm * .html 这样的命令，这会删除当前目录的所有文件。
```

**命令的4中形式**

1. executable file, 可以是二进制的，也可以是文本形式的脚本；
2. builtins, 即 /bin 下面的 [, echo, kill, pwd, test 和 /usr/bin 下面的 alias, bg, cd, command, false, fc, fg, getopts, hash, jobs, printf, read, true, type, ulimit, umask, unalias, wait, which；
3. shell 函数
4. alias 定义的别名

**type**

type是一个极其有用的命令，能立马找到当前环境能用的命令是源自哪里，是什么类型的。示例：

> ➜  ~ type Python3   
Python3 is /Users/didi/.bin/Python3   
➜  ~ ls -l /Users/didi/.bin/Python3  
lrwxr-xr-x  1 didi  staff  22  4 25 10:06 /Users/didi/.bin/Python3 -> /usr/local/bin/python3  
➜  ~ type [  
[ is a shell builtin  
➜  ~ type ls
ls is an alias for ls -G

**where**

where命令的有用之处是它可以列出所有出现的地方，比如电脑里面有两个 git 可执行文件，那么 where git 可以把它们统统列出来。

**man 手册的章节含义**

1. 用户命令
2. 程序接口内核系统调用
3. 库函数程序接口
4. 特殊文件，比如说设备结点和驱动程序
5. 文件格式
6. 游戏娱乐，如屏幕保护程序
7. 其他方面
8. 系统管理员命令

**> file_name**

巧妙的使用重定向，可以将文本文件的内容清空，也可以新建一个空文件。

**重定向的常见用法**

```bash
# 覆盖型重定向 stdout，可以省略那个 1 
ls ~ 1> out.txt 

# 追加性重定向 stdout，可以省略那个 1 
ls ~ 1>> out.txt 

# 覆盖型重定向 stderr 
ls ~ 2> out.txt 

# 追加性重定向 stdout 
ls ~ 2>> out.txt
```

**/dev/null**

随意放东西进去的垃圾箱

**花括号展开**

echo Front-{A..D}--Back 将会输出四项内容，即 {A..D} 表示从 A 到 D 一共四个情况。

**将命令执行结果展开**

```bash
# 只需要把命令放在 $() 里面即可，旧版 shell 也使用把命令放在 `` 中间的方式 
echo $(ls)
```

**引号**

```bash
# 双引号中，参数展开，算术表达式展开，和命令替换仍然有效，比如 
echo "$USER $((2+2)) $(cal)" 

# 单引号中，所有展开都无效
```

**命令行移动光标**

> ctrl a # 移动到行首
> 
> ctrl e # 移动到行末
> 
> alt <- # 左移一个 word
> 
> alt -> # 右移一个 word
> 
> ctrl u # 剪切整行内容
> 
> ctrl k # 剪切光标后面的内容
> 
> ctrl y # 粘贴

**管理进程**

```bash
# 列出终端相关进程
ps 

# 列出所有进程
ps x 

# 列出所有进程的详细信息
ps aux 

# 显示从终端启动的后台进程
jobs 

# 让指定序号的从终端启动的进程返回到前台
fg %JOB_SPEC 

# 让指定序号的从终端启动的进程返回到后台
bg %JOB_SPEC 

# 给进程发送信号，最常见的信号是：
kill -SIG_NUM PID 
    1  HUP  挂起
    2  INT  中断，和 ctrl c 的作用一样
    9  KILL 杀死，这个信号并不会发给进程号是 PID 的进程，而是立即强制停止此进程，被杀进程就没有机会保存数据
    15 TERM kill 命令的默认信号，终止进程，被中止的进程有机会保存数据
```

