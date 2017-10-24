---
layout: post
title:  "iOS逆向系列5_IDA/iFunBox/other"
categories: Reverse
tags: Reverse
author: 3行代码
---

* content
{:toc}

**待整理**

## 1、简介

即使你以前没有从事过iOS逆向工程相关的工作，也一定听说过IDA（The Interactive Disassembler）的鼎鼎大名。而对于绝大多数接触过逆向工程的人来说，IDA三个字则是如雷贯耳，它乃逆向工程中最负盛名的神器之一。如果说class-dump能够帮我们罗列出要分析的点，那IDA就能进一步帮我们把这些点铺成面。

笼统地说，IDA是一个支持Windows、Linux和Mac OS X的多平台反汇编器/调试器，它的功能非常强大，以至于连官方都不能给出一个详尽的功能列表。

IDA的正式版是收费的，但其作者也是程序员出身，深知我们生活不易，所以慷慨地提供了一个免费的试用版，对于逆向工程初学者来说，已完全够用了。

## 2、安装

##### 2.1 安装

IDA的下载和安装十分方便，具体可参考：[https://www.hex-rays.com/products/ida/support/download.shtml](https://www.hex-rays.com/products/ida/support/download.shtml)

##### 2.2 IDA使用说明

启动之后，在该界面中，不用繁琐地在菜单里点击“打开文件”，然后一个目录一个目录地去翻找，只需把要分析的文件拖进IDA的灰色区域就行了。

有一个地方需要注意：对于一些fat binary（指的是为了兼容不同架构的处理器，而把多种指令集糅合到一个binary）来说
![](https://ooo.0o0.ooo/2016/11/01/581869117b6b0.jpeg)

上图最上面的白框内会出现多个Mach-O文件供我们选择。需要我们识别设备对应的ARM信息，例如笔者的iPhone 5对应的是ARMv7s。如果设备的ARM没有出现在这些选项中，就选那个向下兼容的选项，即如果选项里有ARMv7S，就选它；否则选ARMv7。
这里笔者选择了ARMv7S，然后点击“OK”，此时 会连续弹出好几个窗口，一路点击“Yes”和“OK”就可以了。
之后就进入如下界面：
![](https://ooo.0o0.ooo/2016/11/01/58186a17dab43.jpeg)

在上边的界面中，你会看到上方的进度条不断滚动，下方的Output window也会打印出对文件的分析进度。当进度条的主色调变成蓝色（IDA界面上的颜色在黑白印刷页上看不出来，请谅解），Output window中显示“The initial autoanalysis has been finished.”时，表示IDA的初始分析已完成。

在初学阶段，由于主要用IDA作静态分析，基本用不上Output window，所以在开始分析之前，可以先关掉“Output window。

Functions window中的Objective-C函数与class-dump导出的内容吻合。除了Objective-C函数外，IDA还将所有subroutine罗列了出来，这是class-dump做不到的。class-dump导出的内容都是Objective-C函数名，可读性高，容易上手，是iOS逆向工程初学者的乐园；subroutine的名称只是一个代号，没有明显含义，分析难度大，大多数初学者看到这里就打了退堂鼓。但是，iOS的底层是用C和C++实现的，编译之后生成的大都是subroutine，class-dump拿它没辙，只能使用IDA这样的工具。要想深层次挖掘iOS中最有趣的部分，掌握IDA的用法是必经之路。


##### 2.3 Main window

绝大多数没有用过IDA的iOS开发者，包括笔者，在第一次看见初始分析完成后的Main window时都懵了——这都是些什么玩意儿？这是人类文字吗？还是把IDA关了，刷会儿微博压压惊吧。这跟很多工程师在写第一行代码时不知如何下手的感觉很类似。其实，跟写代码时需要定义一个入口函数一样，在逆向工程里，也需要找到自己感兴趣的入口函数。在Functions window中双击这个入口函数，使得Main window跳转到函数体，然后用鼠标选中Main window，按一下空格，界面会瞬间变清爽，可读性一下就强了起来。

。。。


## 3、iFunBox

iFunBox 是一款老牌iOS文件管理工具，可以非常方便地操作iOS中的文件。 iFunBox的使用并不复杂，我们主要用到的是它的文件传输功能。有一点需要注意的是，越狱iOS必须安装“Apple File Conduit 2” ，简称AFC2，这样iFunBox才能够浏览iOS全系统文件，而且这也是接下来大部分操作的先决条件。 

## 4 dyld_decache

安装了iFunBox和AFC2之后，不少读者会迫不及待地开始浏览iOS文件系统，看看这个封闭平台的表面下到底暗藏了多少玄机。相信大家很快就会发现一个问题：“/System/Library/Frameworks/”、“/System/Library/PrivateFrameworks/”等目录下，怎么没有库文件？

从iOS 3.1开始，包括frameworks在内的许多库文件被存进了一个大cache里，这个cache文件位于“/System/Library/Caches/com.apple.dyld/dyld_shared_cache_armx”（名为dyld_shared_cache_armv7、dyld_shared_cache_armv7s或dyld_shared_cache_arm64），可以使用KennyTM开发的dyld_decache将其中的二进制文件提取出来。这样做的好处是确保分析的文件来自本机，在使用Mac工具集与iOS工具集分析同一目标时，OSX与iOS上分析出的指令和地址等数据是完全吻合的，避免了出现驴唇不对马嘴的低级错误。有关这个cache的进一步介绍，可以参阅DHowett的博客http://blog.howett.net/).

使用dyld_decache之前，要将“/System/Library/Caches/com.apple.dyld/dyld_shared_cache_armx”用iFunBox（不能用scp）从iOS拷贝到OSX中。然后从https://github.com/downloads/kennytm/Miscellaneous/dyld_decache[v0.1c].bz2下载dyld_decache。解压之后赋予其可执行权限，如下：

	snakeninnysiMac:~ snakeninny$ chmod +x /path/to/dyld_decache\[v0.1c\]

然后开始提取二进制文件，如下：

```
snakeninnysiMac:~ snakeninny$ /path/to/dyld_decache\[v0.1c\] -o /where/to/store/decached/binaries/ /path/to/dyld_shared_cache_armx
  0/877: Dumping '/System/Library/AccessibilityBundles/AXSpeechImplementation.bundle/AXSpeechImplementation'...
  1/877: Dumping '/System/Library/AccessibilityBundles/AccessibilitySettingsLoader.bundle/AccessibilitySettingsLoader'...
  2/877: Dumping '/System/Library/AccessibilityBundles/AccountsUI.axbundle/AccountsUI'...
……
```

提取出的所有二进制文件都存放在了“/where/to/store/decached/binaries/”下。值得一提的是，逆向工程需要分析的二进制文件现在散落在OSX和iOS两个系统中，不方便查找，建议利用下一章提到的scp工具把iOS文件系统拷贝一份存在OSX里。


 



 