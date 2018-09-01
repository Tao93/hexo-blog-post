---
title: HomeBrew Cask 加快下载的脚本
tags: [Homebrew,Shell,macOS]
---

Homebrew 是个好工具，除了用它安装命令行软件外，我也会用它来安装 GUI 软件。这么做的好处是 Homebrew 会直接帮我接连链接到 GUI 软件的可执行文件的符号链接，这样，我在终端使用符号链接，就可以打 GUI 软件了。例如，Sublime Text 的符号链接是 subl，那么在终端任何目录中，一行 subl "new text.txt" 就可以用 Sublime Text 打开new text.txt了。

问题是，brew cask install xxx 的下载速度经常会非常慢，而我把下载软件包的 URL 直接复制到 Chrome 中去下载，则非常快，这就很尴尬了。经过稍微检查，我发现brew cask install xxx 过程中，会在 Cache/Cask 目录(其实就是 brew --cache命令的结果)中生成一个类似于xxx--v1.2.3.zip.incomplete这样的文件。于是我把自己用 Chrome 下载的文件，复制到 Cache/Cask 中来，然后更名为 xxx--v1.2.3.zip，也就是把 .incomplete 去掉。然后再 brew cask install xxx一下，发现 Homebrew 提示软件包已经下载好了，于是立刻开始安装了。

上面这样的方式确实避免了下载慢的问题，不过这样手动操作，总是不那么高(yǒu)效(bī)率(gé)，所以我就想要写个小脚本来自动化这个过程。我选择了使用 shell 脚本。此处要吐槽一下 shell 脚本的语法，难记，真的是很难记(比如字符串操作)！比起来，几乎任何一门编程语言的语法都要好记得多。

废话说了很多了，下面是脚本内容：

```bash
#!/bin/bash 

if [ "$#" -eq "1" ]; then 
    # 找到本地描述需要安装的软件包的 ruby 文件 
    tap_path=`brew --repo caskroom/cask` 
    file_path="${tap_path}/Casks/${1}.rb" 
    if [ -f "$file_path" ]; then 
        # 从里面读取版本信息，下载的 URL 
        while read key value; do 
            if [[ $key == "version" ]]; then 
                len=$((${#value}-2)) 
                version=${value:1:${len}} 
            elif [[ $key == "url" ]]; then 
                len=$((${#value}-2)) 
                url=${value:1:${len}} 
                url=${url/\#\{version\}/$version} 
                # 使用 Chrome 来下载 
                /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome $url 
            fi 
        done < $file_path 
    else
        echo "Cask '$1' is unavailable: No Cask with this name exists." 
    fi
    cache_dir=`brew --cache`
    downloaded_name=${url##*/}
    target_name=$1--${version}.${downloaded_name##*.} 
    # 每个三秒钟检查一下下载好了没，好了就把下载好的文件移动到 
    # Cache/Cask 中，并且改名为 Homebrew 想要的方式。 
    while true; do 
        if [ -e ~/Downloads/$downloaded_name ]; then 
            mv ~/Downloads/$downloaded_name $cache_dir/Cask/$target_name 
            # 安装 
            brew cask install $1 
            break 
        else
            sleep 3 
        fi 
    done
else
    echo "usage: cask-install Cask-Name" 
fi
```
