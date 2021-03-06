---
layout: post
title:  "iOS逆向系列_综述"
categories:  Reverse
tags:  iOS逆向
author: 3行代码
---

* content
{:toc}


## 前言

**待整理**

未越狱的iOS是个封闭的黑盒子，直到evad3rs、盘古、太极等团队把iOS越狱之后，这个黑盒子才被打开，神秘的iOS得以完整地展现在我们面前。

因为权限极低，来自App Store的普通App（以下简称StoreApp）不能访问自身目录以外的绝大多数文件。而iOS一旦越狱，来自Cydia的App就可以拥有比StoreApp更高的权限，从而访问全系统文件；来自Cydia的iFile即是iOS上一个老牌的第三方文件管理App。

iOS逆向工程，主要作用是：

1、借鉴他人app:想开发IM类APP可以根据竞争对手选择去逆向哪一家的APP，然对手的app整体构架和设计思路对于我们的开发是很有益处的。

2、安全等级鉴定:通过逆向去判别一款软件的安全等级。

## 1、概述

##### 1.1、逆向分为 系统分析+代码分析：

1.1.1 系统分析：

要从整体上观察目标程序的行为特征、文件的组织架构、设计思路，从而为我们做代码分析做准备。

1.1.2 二进制代码分析：

进入代码分析阶段，则要把软件的核心代码还原出来，最终达到我们的目的。

1.1.3 

通过逆向工程，可以推导出某个App的设计思路、内部算法和实现细节，但这是一个非常复杂的过程，可以说是一种解构再重组的艺术。想让自己的逆向工程水平达到艺术的高度，需要对软件开发、硬件原理和iOS系统有透彻的理解。

##### 1.2、分析工具：

相对于正向开发，逆向工程使用的工具并不那么“智能”，很多工作需要我们手工完成。只有对工具的熟练使用，才能够极大地提高逆向工程的效率。iOS逆向工程的工具可以分为四大类：监测工具、反汇编工具（disassembler）、调试工具（debugger）,以及开发工具 。

1.2.1 监测工具

在iOS逆向工程中，起到嗅探、监测、记录目标程序行为的工具统称为监测工具。这些工具通常可以记录并显示目标程序的某些操作，如UI变化、网络活动、文件访问等。iOS逆向常用的监测工具有Reveal(UI变化)、snoop-it、introspy等。

1.2.2 反汇编工具

从UI层面切入代码层面后，就要用到反汇编工具来梳理代码了。反汇编工具把二进制文件作为输入，经过处理后输出这个文件的汇编代码；在iOS逆向工程中，常用的反汇编工具主要是IDA和Hopper。
作为老牌反汇编工具，IDA是逆向工程中最常用的利器之一。它支持Windows、Linux、OSX平台和多种处理器架构。Hopper是一款近年面世的反汇编工具，它主要针对的是苹果系操作系统。下文主要使用IDA

1.2.3 调试工具

iOS开发者对调试工具应该不陌生，在App开发中，少不了在Xcode中调试代码。我们可以在某一行代码上设置断点，使进程能够停止在那一行代码上，并实时显示进程当前的状态。在iOS逆向工程中，用到的调试工具主要是LLDB。

1.2.4 开发工具

从UI层面切入代码层面，用反汇编工具和调试工具分析过二进制文件后，就可以整理分析结果，用开发工具写程序了。对于App开发者来说，Xcode是最常用的开发工具。但是我们一旦将战场从AppStore转移到越狱iOS，开发工具的种类就得到了扩充，不但有基于Xcode的iOSOpenDev，还有偏命令行的Theos。从个人体验来说，Theos是让笔者最为兴奋的开发工具，在知道Theos之前，笔者感觉自己一直都被限制在AppStore中，直到掌握了Theos的用法，才突破了AppStore，完整地认识了整个iOS系统。这里主要以Theos作为开发工具 。

之后系列文章将按照如下的几大块来进行：

OSX 工具集：dump、Theos、Reveal、IDA、iFunBox、dyld_decache

 

IOS 工具集：CydiaSubstrate、Cycript、LLDB与debugserver、dumpdecrypted、OpenSSH、usbmuxd、iFile、MTerminal、syslogd to/var/log/syslog

理论篇：OC理论基础、ARM汇编相关的理论基础、

实战篇：



## 2、IOS 文件系统

因为权限极低，来自App Store的普通App（以下简称StoreApp）不能访问自身目录以外的绝大多数文件。而iOS一旦越狱，来自Cydia的App就可以拥有比StoreApp更高的权限，从而访问全系统文件；来自Cydia的iFile即是iOS上一个老牌的第三方文件管理App。

##### 2.1、IOS目录结构

iOS是由OSX演化而来的，而OSX则是基于UNIX操作系统的。这三者虽然有很大区别，但它们血脉相连。从Filesystem Hierarchy Standard和hier(7)中，可以一窥iOS目录结构的设计标准。

Filesystem Hierarchy Standard（以下简称FHS）为类UNIX操作系统的文件目录结构制定了一套标准，它的初衷之一是让用户预知文件或目录的存放位置。OSX在此基础上形成了自己的hier(7)框架。

越狱后的文件展示:
![](https://ooo.0o0.ooo/2016/11/12/5827229be3e42.png)

类UNIX操作系统的常见目录结构如下所示:

```

·/：根目录，以斜杠表示，其他所有文件和目录在根目录下展开。
·/bin：“binary”的简写，存放提供用户级基础功能的二进制文件，如ls、ps等。
·/boot：存放能使系统成功启动的所有文件。iOS中此目录为空。
·/dev：“device”的简写，存放BSD设备文件。每个文件代表系统的一个块设备或字符设备，一般来说，“块设备”以块为单位传输数据，如硬盘；而“字符设备”以字符为单位传输数据，如调制解调器。
·/sbin：“system binaries”的简写，存放提供系统级基础功能的二进制文件，如netstat、reboot等。
·/etc：“Et Cetera”的简写，存放系统脚本及配置文件，如passwd、hosts等。在iOS中，/etc是一个符号链接，实际指向/private/etc。
·/lib：存放系统库文件、内核模块及设备驱动等。iOS中此目录为空。
·/mnt：“mount”的简写，存放临时的文件系统挂载点。iOS中此目录为空。
·/private：存放两个目录，分别是/private/etc和/private/var。
·/tmp：临时目录。在iOS中，/tmp是一个符号链接，实际指向/private/var/tmp。
·/usr：包含了大多数用户工具和程序。/usr/bin包含那些/bin和/sbin中未出现的基础功能，如nm、killall等；/usr/include包含所有的标准C头文件；/usr/lib存放库文件。
·/var：“variable”的简写，存放一些经常更改的文件，比如日志、用户数据、临时文件等。其中/var/mobile和/var/root分别存放了mobile用户和root用户的文件，是重点关注的目录。

```

作为iOS开发者，日常操作所对应的功能模块大多来自iOS的独有目录：
![](https://ooo.0o0.ooo/2016/11/12/5827237fb21fd.png)

```
·/Applications：存放所有的系统App和来自于Cydia的App，不包括StoreApp

·/Developer：如果一台设备连接Xcode后被指定为调试用机，Xcode就会在iOS中生成这个目录，其中会含有一些调试需要的工具和数据

·/Library：存放一些提供系统支持的数据，其中/Library/MobileSubstrate下存放了所有基于CydiaSubstrate（原名MobileSubstrate）的插件。

·/System/Library：iOS文件系统中最重要的目录之一，存放大量系统组件
·/System/Library/Frameworks和/System/Library/PrivateFrameworks：存放iOS中的各种framework
·/System/Library/CoreServices里的SpringBoard.app：iOS桌面管理器（类似于Windows里的explorer），是用户与系统交流的最重要中介。
/System目录下的玄机远不止上面提到的3个目录这么简单

·/User：用户目录，实际指向/var/mobile，其目录结构如下。这个目录里存放大量用户数据，比如：
     ·/var/mobile/Media/DCIM下存放照片；
     ·/var/mobile/Media/Recordings下存放录音文件；”
     “·/var/mobile/Library/SMS下存放短信数据库；
      ·/var/mobile/Library/Mail下存放邮件数据。
另外一个非常重要的子目录是/var/mobile/Containers，存放StoreApp。值得注意的是，App的可执行文件在bundle与App中的数据目录被分别存放在/var/mobile/Containers/Bundle和/var/mobile/Containers/Data这两个不同目录下

在逆向初期，我们最好关注以下几个文件夹：
·/Developer
·/Library/Logs
·/Library/MobileDevice
```

#### 2.2、 iOS文件权限简介

iOS是一个多用户操作系统。

“用户”是一个抽象的概念，它代表对操作系统的所有权和使用权。比如，mobile用户无法调用reboot命令重启iOS，而root用户却可以；

“组”是用户的一种组织方式，一个组可以包含多个用户，一个用户也可以属于多个组。iOS中的每个文件都有一个属主用户和一个属主组，或者说这个用户和这个组拥有这个文件；每个文件都具有一系列权限，简单地说，权限的作用在于说明文件的属主用户能做什么，属主组能做什么，以及其他所有人能做什么。

iOS用3位（bit）来表示文件的权限，从高位到低位分别是r（read）权限、w（write）权限，以及x（execute）权限。文件与用户的关系存在以下三种可能性：

- ·此用户是属主用户；
- ·此用户不是属主用户，但在属主组里；
- ·此用户既不是属主用户，又不在属主组里。

#### 2.3、 iOS二进制文件类型

在iOS逆向工程初学阶段，我们的目标主要是Application、Dynamic Library（以下简称dylib）和Daemon这三类二进制文件，对它们的了解越深入，逆向工程就会越顺利。这三类文件分工不同，其目录结构和文件权限也有一些区别。

###### 2.3.1 Application

Application就是我们最熟悉的App了。了解下面的几个App相关概念，是开始逆向工程前的必备工作。

1.bundle

bundle的概念来源于NeXTSTEP，它不是一个文件，而是一个按某种标准结构来组织的目录，其中包含了二进制文件及运行所需的资源。正向开发中常见的App和framework都是以bundle的形式存在的；在越狱iOS中常见的PreferenceBundle，可以看成是一种依附于Settings的App，结构与App类似，本质也是bundle。 Framework也是bundle，但framework的bundle中存放的是一个dylib，而不是可执行文件。相对来说，framework的地位比App更高，因为一个App的绝大多数功能都是通过调用framework提供的接口来实现的。将某个bundle确立为逆向目标后，绝大多数逆向线索都可以在bundle内找到，这大大降低了逆向工程的复杂度。

2.App目录结构

在iOS逆向工程中，对App目录结构的熟悉程度是决定工作效率的重要因素。App目录的以下三个部分比较重要。

```
1、Info.plist
Info.plist记录了App的基本信息，如bundle identifier、可执行文件名、图标文件名等。其中bundle identifier会在CydiaSubstrate中成为tweak的重要配置信息。可以通过Xcode查看它的值；Xcode自带的命令行工具plutil查看它的值:
如下： 
plutil -p  目录 | grep CFBundleIdentifier

$ plutil -p /Users/Test/AlipayWallet9.9.5_/Payload/AlipayWallet.app/info.plist | grep CFBundleIdentifier

结果:   "CFBundleIdentifier" => "com.alipay.iphoneclient"

2、可执行文件
可执行文件的重要性不言而喻，它是App目录下最核心的部分，也是逆向工程最主要的目标。同样可以通过Xcode和plutil两种方式来查看Info.plist，定位可执行文件。
 plutil -p  目录 | grep  CFBundleExecutable
 
$ plutil -p /Users/Test/AlipayWallet9.9.5_/Payload/AlipayWallet.app/info.plist | grep CFBundleExecutable

结果：  "CFBundleExecutable" => "AlipayWallet"

3、·lproj目录 本地化
lproj目录下存放的是各种本地化的字符串（.strings），是iOS逆向工程的重要线索，也可以用plutil查看，如下：

$ plutil -p /Users/Test/AlipayWallet9.9.5_/Payload/AlipayWallet.app/en.lproj/Localizable.strings

结果： file does not exist or is not readable or is not a regular file (Error Domain=NSCocoaErrorDomain Code=260 "The file “Localizable.strings” couldn’t be opened because there is no such file." UserInfo={NSFilePath=/Users/Test/AlipayWallet9.9.5_/Payload/AlipayWallet.app/en.lproj/Localizable.strings, NSUnderlyingError=0x7fd538c00b40 {Error Domain=NSPOSIXErrorDomain Code=2 "No such file or directory"}})

```


3.系统App VS.StoreApp

/Applications/目录存放系统App和从Cydia下载的App（我们把来自Cydia的App视为系统App），而/var/mobile/Containers/目录存放的则是StoreApp。虽然两者都是App，但它们在如下方面存在着一些差别。

·目录结构
     两种App的bundle内部目录结构区别不大，都含有Info.plist、可执行文件、lproj目录等，但是数据目录的位置不同：StoreApp的数据目录在/var/mobile/Containers/Data/下，以mobile权限运行的系统App的数据目录在/var/mobile/下，而以root权限运行的系统App的数据目录在/var/root/下。
    
·安装包格式与权限
     Cydia App的安装包格式一般是deb，StoreApp的安装包格式一般是ipa。其中deb是来自Debian的安装包格式，由Cydia作者saurik移植到iOS中，它的属主用户和属主组一般是root和admin，能够以root权限运行；而ipa是苹果为iOS推出的专属App安装包格式，属主用户和属主组都是mobile，只能以mobile权限运行。
     
·沙盒（sandbox）
     通俗地说，iOS中的沙盒就是一种访问限制机制，我们可以把它看作是权限的一种表现形式，授权文件（entitlements）也是沙盒的一部分。它是iOS最核心的安全组件之一，其实现很复杂，这里不过多讨论其细节。总的来说，沙盒会将App的文件访问范围限制在这个App内部，一个App一般不知道其他App的存在，更别说访问它们了；沙盒还会限制App的功能，例如对iCloud接口的调用就必须经过沙盒的允许。
     

###### 2.3.2 Dynamic Library

大部分iOS开发者的日常工作应该都是写App，估计很少有人写过dylib，因此对dylib的概念很陌生。殊不知，在Xcode工程里导入的各种framework，链接的各种lib，其实本质都是dylib。 可以用“file”命令验证一下，如下：
```
$ file /Users/…./Lib/AliPaySDK/AlipaySDK.framework/AlipaySDK
.../Lib/AliPaySDK/AlipaySDK.framework/AlipaySDK: Mach-O universal binary with 4 architectures.../Lib/AliPaySDK/AlipaySDK.framework/AlipaySDK (for architecture i386): Mach-O object i386.../Lib/AliPaySDK/AlipaySDK.framework/AlipaySDK (for architecture x86_64): Mach-O 64-bit object x86_64.../Lib/AliPaySDK/AlipaySDK.framework/AlipaySDK (for architecture armv7): Mach-O object arm
.../Lib/AliPaySDK/AlipaySDK.framework/AlipaySDK (for architecture arm64): Mach-O 64-bit object
```

如果把焦点转移到越狱iOS中，Cydia里的各种tweak无一不是以dylib的形式工作的，正是这些tweak的存在让我们能够随意定制自己的iOS。在逆向工程中，我们会频繁接触各种dylib，因此有必要了解一些相关知识。

在iOS中，lib分为static和dynamic两种

--其中static lib在编译阶段成为App可执行文件的一部分，会增加可执行文件的大小。因为App尺寸变大，启动时需要加载的内容变多，所以可能会导致App启动变慢。

--dylib则相对“智能”一些，它不会改变可执行文件的大小，只有当App需要用到这个dylib时，iOS才会把它加载进内存，成为App进程的一部分。是逆向工程的重要目标类型，但其本身并不是可执行文件，不能独立运行，只能为别的进程服务，而且它们寄生在别的进程里，成为了这个进程的一部分。

因此，dylib的权限是由它寄生的那个App决定的，同一个dylib寄生在系统App和StoreApp里时的权限是不同的。例如，你写了一个Instagram的tweak，用来把喜欢的图片保存在本地，如果保存目录是/var/mobile/Containers/Data/下App对应的Documents目录，那么因为Instagram是一个StoreApp，这样的操作是没有问题的，tweak能够正常工作。而如果保存目录是/var/mobile/Documents，那么在兴高采烈地保存了一大堆美图，准备回头细细品味时，你就会发现/var/mobile/Documents里啥图片也没有——操作都被sandbox给禁掉了。


###### 2.3.3 Daemon


1）当我们正在用iPhone上网或刷微博时来了一个电话，所有其他操作会立即中断，iOS第一时间将接听电话的界面呈现在我们面前。如果iOS中没有真正的后台多任务，系统是如何实时处理这个来电的呢？

2）对于那些经常收到垃圾短信和骚扰电话的”“朋友来说，类似于SMSNinja这样的防火墙软件必不可少。如果它不能常驻iOS后台，怎么能够实时地处理并过滤收到的每一条短信呢？

….
     这些现象无一不说明iOS实际上存在真正的后台多任务。对于StoreApp来说，可以把iOS看作是没有真正后台多任务的系统 ，当用户按下home键时，进程就进入后台了，大多数功能都会被暂停。
     
但iOS源于OSX，后者又跟所有类UNIX操作系统一样，有daemon（即守护进程，Windows称Service）的概念。越狱开放了iOS全文件系统，daemon也得以展现在我们面前。 Daemon为后台运行而生，给用户提供了各种“守护”，如imagent保障了iMessage的正确收发,mediaserverd处理了几乎所有的音频、视频，syslogd则用于记录系统日志等。

iOS中的daemon主要由一个可执行文件和一个plist文件构成。iOS的根进程是launchd，它会在开机时检查/System/Library/LaunchDaemons和/Library/LaunchDaemons下所有格式符合规定的plist文件，然后启动对应的daemon。这里的plist文件与App中的Info.plist文件作用类似，即记录daemon的基本信息，如下：

	$ plutil -p /Users/snakeninny/Code/iOSSystemBinaries/ 8.1.1_iPhone5/System/Library/LaunchDaemons/com.apple.imagent.plist
。。。。。。

相对于App，daemon提供的功能要底层得多，逆向难度也要大得多，随意改动造成的后果当然也就严重得多，所以白苹果的惨案才会时有发生。在iOS逆向工程初学阶段，请不要把daemon当作练习目标；当你逆向了几个App，有了一定的心得和积累后再挑战这些daemon才是比较明智的选择。相比App，逆向daemon花费的时间和精力会更多，但更多的付出一定会带来更丰厚的回报。例如，“iOS上的第一款电话录音软件”Audio Recorder就是通过逆向mediaserverd这个daemon实现的。”




