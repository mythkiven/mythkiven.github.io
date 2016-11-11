---
layout: post
title:  "Sublime Text基础教程"
categories: Sublime
tags:  Sublime 
author: 3Code
---

* content
{:toc}

![](https://ooo.0o0.ooo/2016/10/29/58147e0b8e24a.jpg)

## 说明
**本人目前sublime2\3都在使用，以下涵盖两个版本的内容，环境是OS X EI。建议大家使用sublime3，下边所述部分插件在2上可能有差异。**

## 1、简介

Sublime Text 是一个代码编辑器，也是HTML和散文先进的文本编辑器。Sublime Text是由程序员Jon Skinner于2008年1月份所开发出来，它最初被设计为一个具有丰富扩展功能的Vim。
Sublime Text具有漂亮的用户界面和强大的功能，例如代码缩略图，Python的插件，代码段等。还可自定义键绑定，菜单和工具栏。Sublime Text 的主要功能包括：拼写检查，书签，完整的 Python API ， Goto 功能，即时项目切换，多选择，多窗口等等。Sublime Text 是一个跨平台的编辑器，同时支持Windows、Linux、Mac OS X等操作系统。



## 2、安装


Sublime 的安装比较简单，直接去官网[http://www.sublimetext.com/](http://www.sublimetext.com/)，点击Download菜单，进入之后选择自己操作系统的进行下载安装安装即可。

##### 2.1 命令模式

习惯了Unix系列操作系统的人往往会觉得过于可视化操作会显得很low。当然Sublime Text 提供的也有命令模式操作，但Sublime Text的命令模式要比VIM的好用的多。

cmd+shift+p来打开命令模式：
在弹出的输入框里输入我们需要的命令，比如，我想拷贝当前文件的路径，输入copy之后，选择File:copy path选项之后，当前文件的路径就已经复制到了系统的剪切板上。

Sublime Text的命令模式支持模糊匹配，输入cp回车后可以直接实现上面的拷贝当前文件的路径功能（因为cp模糊匹配了File:copy path）。


##### 2.2 Goto AnyThing [cmd+p]

1、查找文件：如果项目文件目录很多层，文件查找是一个很头疼的问题，不过Sublime Text里有一个叫做Goto AnyThing的功能，通过快捷键cmd+p打开Goto AnyThing窗口，在输入框中输入我们想要打开的文件模糊名称即可，Sublime Text会为我们查找出符合的文件。

2、快速查看文件内部结构：cmd+p打开Goto AnyThing窗口后，输入@字符，就会出现当前文件的结构，如js文件会列出所有方法，md文件会列出大纲。

##### 2.3 安装PackageControl组件

在sublime里面使用快捷键：ctrl+~ 调出控制台，Sublime Text2粘贴如下代码：

```
import urllib2,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation')
```

Sublime Text3可以粘贴如下代码：

```
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
保险起见最好去官网复制代码。[传送门](https://packagecontrol.io/installation#st3)
粘贴回车就会自动安装。成功安装后，会有如下效果:

![](https://ooo.0o0.ooo/2016/10/29/58147bf453227.jpg)

使用 cmd + Shift + P 调出面板，然后输入 pci ，选中“Package Control: Install Package”并回车，然后通过输入插件的名字找到插件并回车安装即可。

**注意的是，需要本地装有python环境.**

有了package control，就可以安装想要的插件了!sublime下面有很多插件，一般编辑器有的，sublime都会以插件的形式出现。之后的操作就类似了，安装某插件xxx:

cmd+Shift+P调出面板 -> 输pci回车 -> 然后搜索自己需要的插件安装即可.

## 进阶

[传送门](http://3code.info/2016/10/30/SublimeUpgrade/)
