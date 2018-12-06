---
layout: post
title:  "使用USTC镜像安装Homebrew以及开源软件"
date:   2018-12-06 19:06:20 +0800
author: bruinspaw 
categories: System 
---

# Hombrew简介

使用过Debian或者Ubuntu的用户能够方便的通过apt-get包管理器来安装开源软件。Mac OS X基于Unix，但是没能提供类似的工具，不能说不是一大缺憾。第三方的包管理器有Flink，MacPorts和Homebrew。这里介绍的是后起之秀：[Hombrew](https://brew.sh)-The missing package manager for macOS。

手工源代码编译的几个难题：
* 解决软件包之间的依赖关系；
* 解决软件包在各平台之间的移植（有时候需要各种补丁）；
* 编译时的选项，等等。

Homebrew可以解决这些难题，并且具有较高的定制性，也就是用户可以修改formulae。目前发现的几个问题：
* Homebrew有些软件为了与系统自带命令工具区分，在部分工具加上了前缀或后缀，有些可以通过--with-default-names恢复原名，但像GNU CoreUtils却被强制加了’g’前缀，无法修改，给使用上带来不便。有些formula的编译选项不是用户所需要的。这些可以通过修改formula解决。
* Homebrew源代码编译是在/private/tmp中进行的，编译完成或中断都会立刻清除，如果由于错误中断的话，那么下次编译又要重新开始，比较耗费时间。
* 编译过程不能使用多核，如make -j2.

# Homebrew安装与使用

在终端中输入：
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
```
该官方安装方法由于网络原因很慢且极易中断；可以修改安装脚本，改用中科大的镜像源安装：
```
$ curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install >> install
```

修改脚本中的GitHub源为中科大开源镜像源，修改前：
```
BREW_REPO = “https://github.com/Homebrew/brew".freeze
```
修改后：
```
BREW_REPO = "https://mirrors.ustc.edu.cn/brew.git".freeze
```

同时将下面这行注释掉，否则安装brew之后，会进行升级，homebrew-core此时仍是GitHub源（即使修改脚本中的 CORE_TAP_REPO变量也是采用GitHub源，脚本中的unless can_macos_use_tls12? … end模块不会执行，见注1安装错误提示）：
```
system "#{HOMEBREW_PREFIX}/bin/brew", "update", "--force"
```

brew安装完成之后，下面两个目录并没有建立，需手动建立，然后拉取相应的repository：
* /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core（也即$(brew --repo hombrew/core）
* /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask（也即$(brew --repo hombrew/cask）

```
$ mkdir -p $(brew --repo homebrew/core)
$ git clone https://mirrors.ustc.edu.cn/homebrew-core.git $(brew --repo homebrew/core)
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core'...
remote: Counting objects: 549277, done.
remote: Total 549277 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (549277/549277), 172.90 MiB | 4.55 MiB/s, done.
Resolving deltas: 100% (351836/351836), done.
$ mkdir -p $(brew --repo homebrew/cask)
$ git clone https://mirrors.ustc.edu.cn/homebrew-cask.git $(brew --repo homebrew/cask)
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask'...
remote: Counting objects: 335684, done.
remote: Total 335684 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (335684/335684), 131.25 MiB | 4.98 MiB/s, done.
Resolving deltas: 100% (237756/237756), done.
```

现在安装完成，下面三个目录均采用中科大开源镜像的源：
* $(brew --repo)
* $(brew --repo homebrew/core)
* $(brew --repo homebrew/cask)

注1： 安装错误提示
```
==> Tapping homebrew/core
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core'...
fatal: unable to access 'https://github.com/Homebrew/homebrew-core/': transfer closed with outstanding read data remaining
Error: Failure while executing; `git clone https://github.com/Homebrew/homebrew-core /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core --depth=1` exited with 128.
Error: Failure while executing; `/usr/local/bin/brew tap homebrew/core` exited with 1.
Failed during: /usr/local/bin/brew update --force
```

安装之后将使/usr/local目录能够被管理员组admin读写。

# 替换Homebrew Bottles源

Homebrew主要分两部分：git repo（位于GitHub）和二进制bottles（位于bintray），替换Homebrew Bottles源为中科大开源镜像：

对于bash用户：
```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

对于zsh用户
```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

# 通过Homebrew安装工具

源代码临时目录在~/Library/Caches/Homebrew下，安装后不会删除，可以手动删除。
安装前可以通过命令查看工具：
```
brew info <formulae>
```

## 安装CoreUtils
默认安装会给程序名加前缀g，且该安装没有提供--with-default-names选项，因此需要通过修改coreutils.rb脚本来编译安装，去掉前缀和bottle相关的内容：
```
brew edit coreutils
```
然后安装（选项-v为显示编译过程）：
```
brew install coreutils -v
```

## 安装FindUtils, Grep
默认安装会给程序名加前缀g，但该安装没有提供--with-default-names选项，因此可以运行（也是编译安装）：
```
brew install findutils --with-default-names -v
brew install grep --with-default-names -v
```

## 安装Gawk
awk.rb不是GNU版本的，此处安装gawk：
```
brew install gawk
```

## 安装Sed
默认的脚本是gnu-sed.rb，可以拷贝该脚本并将类名由GnuSed修改为Sed，安装会给程序名加前缀g，但该安装没有提供--with-default-names选项：
```
brew install sed --with-default-names -v
```

## 安装Indent
默认的脚本是gnu-indent.rb，可以拷贝该脚本并将类名由GnuIndent修改为Indent，安装会给程序名加前缀g，但该安装没有提供--with-default-names选项：
```
brew install indent --with-default-names -v
```

## 安装Tar
默认的脚本是gnu-tar.rb，可以拷贝该脚本并将类名由GnuTar修改为Tar，安装会给程序名加前缀g，但该安装没有提供--with-default-names选项：
```
brew install tar --with-default-names -v
```

## 安装Bison和Flex
```
brew install bison flex
```
需要手动在/usr/local/bin中建立它们的连接：
```
$ ln -s /usr/local/Cellar/bison/3.2.2/bin/bison /usr/local/bin/bison
$ ln -s /usr/local/Cellar/flex/2.6.4/bin/flex /usr/local/bin/flex
```

## 安装cmake，tree，unrar，wget
```
brew install cmake tree unrar wget
```

## 安装Python3
```
brew install python
```
将自动安装Python3和pip3
通过pip3 install \<package\>安装到/usr/local/lib/python3.7/site-packages

## 安装Jupyter
```
pip3 install jupyter
```

## 安装Python2
```
brew install python@2
```
将自动安装Python2和pip
通过pip install \<package\>安装到/usr/local/lib/python2.7/site-packages
