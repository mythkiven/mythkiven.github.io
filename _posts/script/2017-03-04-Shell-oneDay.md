---
layout: post
title:  "Shell 脚本"
categories:   脚本
tags: shell
author: 3行代码
---

* content
{:toc}

>
> 作为一名 iOS 老司机,当然要得拿下脚本噻,除了 python, 在 mac 上最常用的就是 shell 脚本了.
>
> 以前是看过但用的少,几年了都忘了差不多了,今天温习一遍,从最简单的开始.
>

说起脚本,就得说脚本的通用性,说起通用性,就得说常见的操作系统: OS X \ Linux \ Windows \ Unix
简单介绍下异同:

- BellLAB 发明了 Unix 和 C ;
- Unix 的 GUI 是 X Window(在内核外跑的应用),包括X Server(与内核通信)与X Client(负责显示)两部分;
- OS X 的 GUI 是自己开发的 Aqua(嵌入内核中) ,而内核则是基于Darwin(Unix + BSD);
- OS X 是全新设计的,和 OS (Mach内核)不同, OS 不是基于 Unix 的.
- Linux 一开始就是免费开源的,基于 Unix ;

以上是 Unix 派的,下边的 Windows 则是 DOS NT 系的:

- Windows 则是 DOS + NT 内核;
- Windows 使用反斜杠: \ 分割路径,原因在于其来自 DOS 最初支持软盘,使用斜杠: /  表示命令参数起始符号
- Unix 使用斜杠:/分割路径

扯的有点远了...

下边回归正文:

### 热身小例子

``` shell
#!/bin/sh                # 制定脚本解释器,使用 /bin/sh  作为解释器
cd ~                     # 切换到当前用户的 home 目录
mkdir shell_txt          # 创建一个 shell_txt 文件夹
cd shell_txt             # 进入 shell_txt 文件夹

for ((i=0; i<10; i++)); do     # 循环10次
    touch txt_$i.txt           # 创建10个文件:test_1…10.txt文件
done                           # 结束循环

```

- 上边的脚本就创建了 shell_txt 文件夹,并创建10个 txt 文件.
-  cd, mkdir, touch都是系统自带的程序，一般在/bin或者/usr/bin目录下
- for, do, done是sh脚本语言的关键字。

### shell 概念

上边是入门热身,下边解释概念:

> 1. shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务
>
> 2. shell脚本（shell script），是一种为 shell 编写的脚本程序
>
> 3. 业界所说的shell通常都是指shell脚本
> 
> 4. shell编程跟java、php编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。
> 
> 5. 当前主流的操作系统都支持shell编程

- Linux 默认安装就带了 shell 解释器
- Mac OS 不仅带了 sh、bash 这两个最基础的解释器，还内置了 ksh、csh、zsh 等不常用的解释器。
- windows 出厂时没有内置 shell 解释器，需要自行安装

### 脚本解释器

- sh   即Bourne shell，POSIX 标准的shell解释器，它的二进制文件路径通常是/bin/sh，由Bell Labs开发

- bash  是Bourne shell的替代品，属GNU Project，二进制文件路径通常是/bin/bash。

- 业界通常混用bash、sh、和shell，比如你会经常在招聘运维工程师的文案中见到：熟悉Linux Bash编程，精通Shell编程


### 脚本语言

理论上讲，只要一门语言提供了解释器（而不仅是编译器），这门语言就可以胜任脚本编程，常见的解释型语言都是可以用作脚本编程的，如：Perl、Tcl、Python、PHP、Ruby。Perl是最老牌的脚本编程语言了，Python这些年也成了一些linux发行版的预置解释器。

编译型语言，只要有解释器，也可以用作脚本编程，如C shell是内置的（/bin/csh），Java有第三方解释器Jshell，Ada有收费的解释器AdaScript。

shell 只定义了一个非常简单的编程语言，如果你的脚本程序复杂度较高，或者要操作的数据结构比较复杂，那么还是应该使用Python、Perl这样的脚本语言，或者是你本来就已经很擅长的高级语言。因为sh和bash在这方面很弱，比如说：

如果你的脚本是提供给别的用户使用，使用sh或者bash，你的脚本将具有最好的环境兼容性，perl很早就是linux标配了，python这些年也成了一些linux发行版的标配，至于mac os，它默认安装了perl、python、ruby、php、java等主流编程语言。



### 编写脚本


打开文本编辑器，新建一个文件，扩展名为sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用php写shell 脚本，扩展名就用php好了。

输入一些代码，第一行一般是这样：

``` shell 
#!/bin/bash
#!/usr/bin/php
```

“#!”是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行。

### 运行脚本

运行Shell脚本有两种方法：

- 作为可执行程序

``` shell 
chmod +x test.sh
./test.sh
```

注意，一定要写成./test.sh，而不是test.sh，运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

通过这种方式运行bash脚本，第一行一定要写对，好让系统查找到正确的解释器。

这里的"系统"，其实就是shell这个应用程序（想象一下Windows Explorer），但我故意写成系统，是方便理解，既然这个系统就是指shell，那么一个使用/bin/sh作为解释器的脚本是不是可以省去第一行呢？是的。

- 作为解释器参数

在终端,知己运行解释器 :

``` shell 
$/bin/sh test.sh
$/bin/php test.php
```

这种方式运行的脚本，不需要在第一行指定解释器信息.









参考 [30min_guides](https://github.com/qinjx/30min_guides/blob/master/shell.md)




