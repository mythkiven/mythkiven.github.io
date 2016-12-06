---
layout: post
title:  "IOS逆向开发系列_Reveal_解码任意APP的UI界面"
categories:  IOS逆向开发
tags:     Reveal iOSOpenDev
author: 3行代码
---

* content
{:toc}

## 前言

从最开始写在笔记上的Reveal使用记录，到现在开放到博客，Reveal也历经了好几个版本：1.5.x->1.6.x->V++(V2等)。
下边更新了以前写的记录，最近安装Reveal的朋友可以参照此教程。

## 关于破解：

如果是使用1.6.3及以下的版本，可以google到破解文件，覆盖安装就行。
如果是V2及以上版本，破解方法我还没找呢，公司用的是1.6.3版本，家里的就通过修改电脑时间。。
顺便说下，免费使用时长，是可以通过修改电脑日期来调整。这个是我用过的版本里通用的福利方法。。

新版的Reveal上缺少libReaveal.dylib，可以基于Reveal.framwwork使用iOSOpenDev创建一个。

## RevealLoader

在手机上添加源:http://rheard.com/cydia

然后安装 Reveal Loader。注意需要VPN才行。

## SSH

手动开启，或者使用PP助手，没有使用过的话，记得开启后修改密码。

## Reveal

与其他几个功能相近的工具比如PonyDebugger（https://github.com/square/PonyDebugger）和Spark Inspector（http://sparkinspector.com）相比，最明显的区别就是直观。

其常规用法是将framework集成至Xcode工程中，具体可参见Reveal的官网[http://revealapp.com/](http://revealapp.com/)，

下面进重点。

## framework +  dylib + plist

####  RevealServer.framework
通过openSSH拷贝framework到越狱机

    scp -r /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework root@iphoneip:/System/Library/Frameworks  

早期版本(V++之前)的libReveal.dylib是支持ARM架构的，那时，只要把这个libReveal.dylib文件扔到手机的/Library/MobileSubstrate/DynamicLibraries/目录下，就OK了，正如下边的已经废弃的代码：

    scp /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib root@iphoneip:/Library/MobileSubstrate/DynamicLibraries  

####  libReveal.dylib

这个文件是没有的，网上有一种方法是使用iOSOpenDev创建的，简述如下：

###### 方法1：iOSOpenDev创建一个

- 1、安装 macports
- 2、 已经安装可能要升级,更新MacPorts到最新版本，时间比较长。 $sudo port -v selfupdate 
- 3、更新MacPorts后，安装DPKG文件，装过的忽略。 $ sudo port -f install dpkg
- 4、安装iOSOpenDev [http://iosopendev.com/download/](http://iosopendev.com/download/)
- 5、期间会各种报错，可以google解决。
- 6、新建CaptainHookTweak工程，使用Reveal.framwwork生成一个libReaveal.dylib即可。
注意一定要添加相关的库，否则编译会报错。

###### 方法2

将文件/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework/RevealServer 重命名为libReveal.dylib copy到手机目录下：/Library/MobileSubstrate/DynamicLibraries/



####  libReveal.plist

在电脑上创建libReveal.plist，然后copy到iphone的 /Library/MobileSubstrate/DynamicLibraries/ 目录下。
libReveal.plist内容如下（Bundles里写要分析的app的Bundle，可以制定多个):

```
{     
    Filter = {    
         Bundles = ("info.3code.www");     
    };     
}  
```

说明：

0、只有安装 Cydia Substrate之后才会有MobileSubstrate目录。mobilesubstrate是cydia插件/软件运行的一个基础依赖包。它提供软件运行的公共库，可以用来动态替换内存中的代码、数据等。安装的插件或软件比如iFile、Activator、SBSettings几乎都是依赖mobilesubstrate才能运行的。

1、libReveal.plist：这个plist是有过滤作用的，它决定你要研究那个App，你通过指定bundleId来决定哪个App/进程启动时，这个libReveal.dylib注入进去。当libReveal.plist不存在时，相当于没有过滤，没有限制，所有进程都会被注入，可能出现白苹果，修复方法见google...

2、手机的/Library/MobileSubstrate/DynamicLibraries目录下存放着所有在系统启动时就需要加载的动态链接库，所以把reveal的动态链接库上传

以上完成之后：

    # killall SpringBoard
    或者手动重启

然后 开启Reveal，就可以尽情的分析APP喽。

链接：
[https://github.com/heardrwt/RevealLoader](https://github.com/heardrwt/RevealLoader)





