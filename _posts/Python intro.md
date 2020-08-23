---
title: High Level Introduction to Python 
tags: [Python]
---

#### 概述

[Python](https://www.python.org/) 诞生自 1991 年，现已在数据科学、Web 开发和软件测试等领域有非常广泛的应用。可能有两方面的原因促使 Python 如此成功，一个是语法简单易于上手，另一个是丰富的第三方扩展包 (Package)。

本文将从高层次角度介绍 Python 的各种重要特性，并横向对比其它语言，从而尽量覆盖核心要点。

#### 解释型与编译型

Python 是解释型语言 ([Interpreted language](https://en.wikipedia.org/wiki/Interpreted_language))，与之相对的是编译型语言 ([Compiled language](https://en.wikipedia.org/wiki/Compiled_language))。

对编译型语言，源代码需要先经过一步或多步编译成另一套格式的二进制文件（可能是可执行文件，也可能是 Java 的字节码之类的），然后二进制文件才能被各个平台 (Windows, macOS 与 Linux 等) 执行。典型的编译型语言有 C/C++, Java 等。

而解释型语言的源代码不需要编译，而是由解释器 ([Interpreter](https://en.wikipedia.org/wiki/Interpreter_(computing))) 直接执行源代码。形象地说，解释器每读一行源代码，就执行一行，直到结束或者发生错误后退出，这种读一行执行一行的行为，类似于老师教学生读书，每读一行文字，老师就向学生解释一行文字，所以称之为解释型。常见的解释型语言有 Python, Perl, JavaScript 等。插播一个笑话，Java 和 JavaScript 的关系，就好比雷锋和雷峰塔的关系。

对 Python 而言，我们写好的代码，就要交给解释器执行了。

#### Python 解释器

Python 解释器，其实就是我们电脑中的一个可执行文件，它在 Windows 中是 python.exe 文件，在类 Unix 系统 (macOS, Linux 等) 中是名为 python (或者 python3) 的文件。有的系统自带 Python 解释器，有的需要安装。有时候一个系统里可能有多个 Python 解释器。我们必须仔细区分，并把我们希望使用的解释器所在的目录路径，放置在 [PATH 环境变量](https://en.wikipedia.org/wiki/PATH_(variable))中，从而直接执行 python 命令时是使用我们期望的那个。

不带参数直接执行 python 命令时，会进入交互模式 (Interactive Mode):

``` bash
bash-3.2$ python

WARNING: Python 2.7 is not recommended. 
This version is included in macOS for compatibility with legacy software. 
Future versions of macOS will not include Python 2.7. 
Instead, it is recommended that you transition to using 'python3' from within Terminal.

Python 2.7.16 (default, Jul  5 2020, 02:24:03) 
[GCC 4.2.1 Compatible Apple LLVM 11.0.3 (clang-1103.0.29.21) (-macos10.15-objc- on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> print 'hello world'
hello world
>>> 
```

交互模式最显著的特征是提示符 `>>>`，就好比 `$` 常常作为 shell 命令前的提示符。交互模式下，每输入完一小段代码并按下回车键，这一小段代码就立即被执行，如果这小段代码有输出，那么相应的输出也会显示出来。这种反复的 `用户输入代码` 与 `解释器执行并输出` 循环，就好比是用户与解释器之间的对话一样，用户每问一句，解释器就答一句，所以称为交互模式。

除交互模式外，还有就是脚本模式。在 python 命令后面接上一个 python 源代码文件的路径，即可以让解释器一次性执行完整个源代码文件。例如编写下面这样一个源代码文件并命名为 `hello_world.py`:

``` python
print 'hello world'
```

以上就是一个最简单的 Python 源代码文件，通过命令 `python <path to hello_world.py>` 即可执行它，如：

``` bash
bash-3.2$ python hello_world.py 
hello world
bash-3.2$ 
```

#### Python 2 与 3

Python 3 是一次不兼容的大升级。这使得按 Python 2 语法所写的代码，无法被 Python 3 解释器正确执行。 有些有大量  Python 2 代码的用户便不愿意为升级到 Python 3 而改代码，所以 Python 3 的推广并不容易。直到  2020 年为止的很长时间内，都是 Python 2 与 3 并存，官方对两者都在维护（甚至都在不断升级）。但 Python 3 终究是大方向，所以新学 Python 的话，按照 Python 3 语法来学习更好。从现在开始，后文的 Python 代码示例按 Python 3 的语法。

一台电脑里可能既有 Python 2 也有 Python 3，甚至它们各自也可能存在不止一份。Python 3 的解释器一般会被命名为 python3，从而和 Python 2 的区分开来。在类 Unix 系统中可以通过 `which -a python` 和 `which -a python3` 来找到当前 PATH 环境变量中能找到的所有 python 解释器。

#### 动态类型

解释型语言往往是动态类型的。所谓动态类型，就是代码中的变量没有固定类型，任何类型的值或者对象都可以赋值给一个变量。

在 Python 中，准确来讲没有东西被称为变量，而只有引用 (reference，有时也称为 name) 和对象。

```python
a + b
```
Python 语言中，上面是一个表达式，它的准确描述是引用 `a` 所指的对象和引用 `b` 所指的对象做一个 `+` 操作。Python 有运算符重载，所以这里 `+` 不一定是算术上的加法。事实上 Python 里面不光有算术加法，字符串对象之间也可以相加，即字符串拼接。

引用是没有类型的，但是引用所指的对象有类型，而通过赋值，一个引用可以指向不同类型的对象，这就是 Python 的动态类型行为。例如，下面的例子中，引用 `a` 先后指向了整数对象和字符串对象：

```python
a = 12
a = "12"
```
Python 中的引用不需要声明，只需要赋值时在等号左边写一个名字，就创造出了引用，后面的代码就可以使用这个引用。这些动态类型的行为，使 Python 代码更加简短简介，但也使得 Python 代码的可读性不如强类型语言。

动态类型的 Python 代码：

```python
def fibonacci(n):
    if (n <= 0):
        return []
    result = []
    for i in range(0, n ):
        if (i == 0 or i == 1):
            result.append(1)
        else:
            result.append(result[i - 1] + result[i - 2])
    return result
```

强类型的 Java 代码：

```java
List<Integer> fibonacci(int n) {
    if (n <= 0) {
        return Collections.emptyList();
    }
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < n; ++i) {
        if (i == 0 || i == 1) {
            result.add(1);
        } else {
            result.add(result.get(i - 1) + result.get(i - 2));
        }
    }
    return result;
}
```

可见 Java 中的各个函数参数、函数返回值和变量都有类型，连列表的元素类型也是明确的，这样的代码更长，写起来要打更多字符，但是可读性更好。相反，Python 就需要良好的命名风格和充足且明确的注释，才能提高代码可读性。

动态类型的解释型语言，常常被称为脚本语言，所以 Python 代码文件会被称为 Python 脚本。

#### 性能与易用性

早期的计算机硬件性能弱，因而编程语言的软件性能很关键，C/C++ 就是高软件性能的代表语言。但随着硬件的快速发展，除游戏和科学计算等领域，程序执行的软件性能对我们不那么重要了，相反我们的软件在不断地变得庞大且复杂。如果一直是用 C/C++ 等 1970 年代诞生的语言，其软件性能对我们并没多大用处，反而是它们的某些特性会对编写复杂的软件形成拖累（比如手动管理内存的分配和释放，手动管理指针，这些可能会成为程序员的噩梦）。

所以进入 1990 年代后，新诞生的编程语言往往会在易用性上下大功夫，让程序员在短短几句代码中就能做很多事情，尽管牺牲了软件性能，但是大部分场景，这种牺牲是非常值得的，在软件日益复杂的情况下，易用性对程序员是很必要的。

使用 C/C++ 就好比使用一把剪刀去修剪草坪，你可以用它做一些比较精细的操作。如果只是 50 平方米的草坪，你可能尚且愿意用剪刀去修剪，但如果是 10 亩草坪，你肯定需要一个割草机，而更易用的语言就好比是割草机。

#### 语句

和很多语言一样，语句 (Statement) 是 Python 代码的基本单位 (比语句更小的是标识符和表达式)。语句有简单语句，也有复合语句，复合语句是简单语句加上循环结构或条件分支组成的。

简单语句：

```python
print('hello world')
```

复合语句：

```python
if (1 + 1 > 2):
    print('impossible')
else:
    print('correct')
```

以上示例也引出一个广为人知的 Python 语法规则，即语句层次的递进，是使用缩进来表示的，整个 Python 文件的缩进必须一致有序，不然会报语法错误。

#### 函数与方法

和很多语言一样，函数 (Fucntion) 与方法 (Method) 是高级编程语言中可被重复调用的代码单元。Python 中直接写在代码文件中的叫做函数，而定义在类 (Class) 中， 可以通过类的对象来调用的，称为方法。

函数：

```python
def fibonacci(n):
    if (n <= 0):
        return 0
    elif n == 1 or n == 2:
        return 1
    else:
        return fibonacci(n - 1) + fibonacci(n - 2)
```

#### 模块

当代码量比较大时，我们往往把代码写在多个代码文件中。当代码文件很多时，我们往往把文件放置在多个目录中，所有目录组成树状结构。

在 Python 中，代码文件称为模块 (Module)，通过 `import` 语句把别的模块导入，即可调用这个被导入模块中的函数或使用它的对象等。

假设有名为 `fibonacci.py` 的文件如下：

```python
def get(n):
    if (n <= 0):
        return 0
    elif n == 1 or n == 2:
        return 1
    else:
        return get(n - 1) + get(n - 2)

```

另有一个名为 `test.py` 的文件内容如下：

```python
import fibonacci

print(fibonacci.get(5))
```

两个文件放置在同一个目录，然后在 Shell 终端进入此目录，并执行 `test.py` 脚本，即可调用到 `fibonacci` module 中的 `get` 函数。

执行到 import 语句时，被导入的 module 里的函数不会马上被执行，但该 module 里的不属于任何函数的语句会立即执行。module 中这样的语句的本意是初始化一个 module。下面给出一个例子，虽然这个例子没有体现初始化的效果，但可以说明逻辑上的事实。在上述 `test.py` 所在目录中新建 `test2.py` 且内容如下:

```python
import test
```

当我们执行 `test2.py` 后，会发现输出了结果 `5`。

#### 包 (Package)

前面说到 Python 代码文件可以放在多层目录结构中，这样的目录中如果有 `__init__.py` 这样一个特殊文件，那么这个目录就被称为包。显而易见 `__init__.py` 是用于初始化一个包，需注意即使我们不需要任何初始化的工作，也要有这个文件才行。

例如前面的例子中，把 `fibonacci.py` 放置在原本所在的目录中的 `fibo` 子目录中，且 `fibo` 子目录中新建空白的 `__init__.py` 文件。那么就可以从包 `fibo` 中导入模块 `fibonacci` 了，即 `test.py` 可以是下面这样：

```python
import fibo.fibonacci

print(fibo.fibonacci.get(5))
```

事实上，除了我们自己写的模块和包，Python 标准库中的代码也是以模块和包的结构组织的。Python 刚被安装好时，这些标准库就存在了。

#### 模块搜索路径

如果一个 Python 文件随意放置在文件系统的某个角落里，那么大概率我们无法成功 import 它。事实上 Python 运行时有个 `sys` 模块，它里面有个 `path` 变量，此变量包含了一串目录路径，这就是解释器搜索模块的整个范围。

以下在交互模式下 `path` 变量的内容:

```bash
Python 3.7.5 (default, Nov  1 2019, 02:16:32) 
[Clang 11.0.0 (clang-1100.0.33.8)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python37.zip', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload', '/usr/local/lib/python3.7/site-packages']
>>> 
```

脚本模式下 `path` 变量的内容:

```bash
['/Users/xxx/Downloads', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python37.zip', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload', '/usr/local/lib/python3.7/site-packages']
```

对比后可见，脚本模式下，脚本所在的目录会插入到 `path` 列表的最开头，也就是有最高的搜索优先级。前文中说到 Python 标准库，事实上标准库的代码文件一般就在 `sys.path` 列表中的某个目录里面。而由于脚本模式下脚本所在的目录具有最高优先级，所以务必注意我们自己写的代码文件的名字，不要和标准库中的代码文件重名，因为重名会导致 import 的时候 import 了我们自己的代码文件，而不是标准库的 module，这往往会导致错误。

#### 扩展包

本文最开始说到，Python 的流行得益于其丰富的扩展包。事实上 Python 安装包本身并不大，这也意味着 Python 标准库的内容是比较有限的，只能做一些比较有限的工作。Python 标准库就好比一台刚买来的新电脑，它已经有了浏览器等软件，但这还不够，我们会安装许多第三方软件来扩展电脑的功能。类似的，可以给 Python 安装各种第三方扩展包。所有安装的扩展包都位于上面 `sys.path` 例子中的最后一个目录中 (我的情况下是 `/usr/local/lib/python3.7/site-packages` 目录)。

电脑中安装软件我们往往会手动去软件官网下载，然后手动安装，必要时手动卸载或者更新。这些手动操作并不方便，而且偶尔可能误入假网站，下载到广告软件甚至包含恶意病毒的软件。如果有一个类似 iOS 上的 App Store 这样一个可靠的官方软件平台，那就再好不过了。Python 还真有这样一个平台，也就是 [Python Package Index](https://pypi.org/)。

在 Python Package Index (以下简称 PyPI) 可以搜索和下载任意扩展包，也可以浏览扩展包的描述文档。然而手动从 PyPI 下载并安装扩展包还是太麻烦了。所以我们需要一款包管理器。

通常而言，包管理器 (Package Manager) 是负责搜索、安装、升级、卸载软件包的命令行工具，如果软件包之间有依赖关系，那么包管理器在安装一个软件包时，被此软件包依赖的所有包也要自动安装好。Python 扩展包的管理，就很需要一款包管理器，事实上有一款名为 `pip` 的 Python 扩展包，就是用于 Python 著名包管理器。

一般 `pip` 都预安装好了，可以直接使用它，顶多它会提示需要升级，只要按提示用它自己升级它就好了。假如 `pip` 还未被安装，那么必须先[手动安装它](https://pip.pypa.io/en/stable/installing.html)，然后才能使用它来安装别的包。这样点像早期的安卓手机，刚买来时可能没有应用商店，用户需要手动先下载并安装一个豌豆荚之类的，才能用豌豆荚去搜索、安装别的 app。

虽然 `pip` 会有可执行文件，让我们能在 shell 终端直接执行 `pip` 命令，但如果电脑中有多个 Python 解释器，建议使用 `python -m pip` 或 `python3 -m pip` 来代替直接使用 `pip`，从而避免混淆。

使用 `pip` 来搜索可以执行 `python3 -m pip search selenium`，安装可以通过执行 `python3 -m pip install selenium`，其它更多用法就不赘述了。

#### 结语

Python 是一门强大而易于上手的语言，丰富的第三方扩展包为 Python 提供了强大的加持，总之这是一门值得学习的语言。限于篇幅，本文几乎不包含 Python 语法方面的知识，而是从一些高层次角度简介了 Python 语言，着重提到了初学者可能不会注意到的要点。至于语法知识，相信在循序渐进的练习中是可以掌握好的。

Python [官方文档](https://docs.python.org/3/)和它里面的 Tutorial 是很不错的参考资料，此外善用 [help](https://docs.python.org/3/library/functions.html#help) 和 [dir](https://docs.python.org/3/library/functions.html#dir) 函数，或者使用 PyCharm 等 IDE 来浏览文档，也是很不错的选择。



