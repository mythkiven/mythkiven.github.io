---
layout: post
title:  "IOS逆向系列_Reveal_解码任意APP的UI界面"
categories:  IOS逆向
tags:  Reveal iOSOpenDev
author: 3行代码
---

* content
{:toc}

## 前言

Reveal可以直观地查看App的UI布局，官方给Reveal的定位是“See your application’s view hierarchy at runtime with advanced 2D and 3D visualisations”，但作为逆向开发，查看自己App的UI布局显然不能满足我们的需求，能够查看别人App的UI布局才是正经事儿。

从使用至今，Reveal也历经了好几个版本：1.5.x->1.6.x->V++(V2等)，不论是哪个版本都有免费使用的机会。旧版的可以直接破解，新版的可以修改系统时间来解决。

## iphone配置

### RevealLoader

1、在手机上添加源:http://rheard.com/cydia然后安装 Reveal Loader。注意需要VPN才行。

2、下载完Reveal Loader后，检查iOS上的/Library/目录下有没有一个名为RHRevealLoader的文件夹:

    # ls -l /Library/ | grep RHRevealLoader

如果没有，就手动创建一个:

    # mkdir /Library/RHRevealLoader

3、Reveal Loader的配置界面位于Settings应用中，点击Reveal进入其界面，呈现在我们面前的主要是一些使用声明。点击Enabled Applications，进入配置界面。要分析哪个App，就打开对应的开关。

## Mac配置

### 1、SSH

手动开启，或者使用PP助手，没有使用过的话，**记得开启后修改密码。**

### 2、关于破解：

如果是使用1.6.x以下的版本，可以google到破解文件，覆盖安装就行。
如果是V2及以上版本，破解方法我还没找，现在用的是1.5.x。。

注意：新版的Reveal缺少libReaveal.dylib，可以基于Reveal.framwwork使用iOSOpenDev创建一个。

### 3、Reveal

与其功能相近的工具有：PonyDebugger（https://github.com/square/PonyDebugger）和Spark Inspector（http://sparkinspector.com）。不过Reveal在UI上更直观些。

其常规用法是将framework集成至Xcode工程中，具体可参见Reveal的官网[http://revealapp.com/](http://revealapp.com/)，

下面进重点。

### 4、framework +  dylib + plist

#### 4.1  RevealServer.framework
通过openSSH拷贝framework到越狱机

    scp -r /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework root@iphoneip:/System/Library/Frameworks  

早期版本的libReveal.dylib是支持ARM架构的，只要把这个libReveal.dylib文件扔到手机的/Library/MobileSubstrate/DynamicLibraries/目录下，就OK了：

    scp /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib root@iphoneip:/Library/MobileSubstrate/DynamicLibraries  

####  4.2 libReveal.dylib

然后打开Reveal，在它标题栏的Help选项下，点开Show Reveal Library in Finder,把libReveal.dylib拷贝到RHRevealLoader目录下。

新版的这个文件是没有的，麻烦点可以使用iOSOpenDev创建：

###### 方法1：iOSOpenDev创建一个

- 1、安装 macports
- 2、 已经安装可能要升级,更新MacPorts到最新版本，时间比较长。 $sudo port -v selfupdate 
- 3、更新MacPorts后，安装DPKG文件，装过的忽略。 $ sudo port -f install dpkg
- 4、安装iOSOpenDev [http://iosopendev.com/download/](http://iosopendev.com/download/)
- 5、期间会各种报错，可以google解决。
- 6、新建CaptainHookTweak工程，使用Reveal.framwwork生成一个libReaveal.dylib即可。
注意一定要添加相关的库，否则编译会报错。

###### 方法2

注意，还有简单的方法😁😁😁

将文件/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework/RevealServer 重命名为libReveal.dylib。copy到手机目录下：/Library/MobileSubstrate/DynamicLibraries/即可


#### 4.3 libReveal.plist

在电脑上创建libReveal.plist，然后copy到iphone的 /Library/MobileSubstrate/DynamicLibraries/ 目录下。

libReveal.plist内容如下（Bundles里写要分析的app的Bundle，可以制定多个):

```
{     
    Filter = {    
         Bundles = ("info.3code.www");     
    };     
}  
```

**注意：**

0、只有安装 Cydia Substrate之后才会有MobileSubstrate目录。mobilesubstrate是cydia插件/软件运行的一个基础依赖包。它提供软件运行的公共库，可以用来动态替换内存中的代码、数据等。安装的插件或软件比如iFile、Activator、SBSettings几乎都是依赖mobilesubstrate才能运行的。

1、libReveal.plist：这个plist是有过滤作用的，它决定你要研究那个App，你通过指定bundleId来决定哪个App/进程启动时，这个libReveal.dylib注入进去。当libReveal.plist不存在时，相当于没有过滤，没有限制，所有进程都会被注入，可能出现白苹果，修复方法见google...

2、手机的/Library/MobileSubstrate/DynamicLibraries目录下存放着所有在系统启动时就需要加载的动态链接库，所以把reveal的动态链接库上传

以上完成之后：

    # killall SpringBoard
    或者手动重启

然后 开启Reveal，就可以尽情的分析APP喽。

链接：
[https://github.com/heardrwt/RevealLoader](https://github.com/heardrwt/RevealLoader)





