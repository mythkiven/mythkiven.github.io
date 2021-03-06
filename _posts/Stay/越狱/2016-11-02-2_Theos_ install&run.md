---
layout: post
title:  "iOS逆向系列2_Theos_安装编译打包运行"
categories: Reverse
tags:  Reverse
author: 3行代码
---

* content
{:toc}

**待整理**

在iosOpenDev之前，很多ios插件都使用theos编译开发，现在使用theos开发的人也不在少数。theos 有自己的模板用于开发一系列的插件程序，所以在早期开发的插件中基本上都是使用theos。


## 1、说明
使用theos创建工程：projectBy3code。文件夹内有四个文件：
projectBy3code.plist、Makefile、Tweak.xm、control 下面分别介绍这四个文件

参考教程：

Logos 语法
[http://iphonedevwiki.net/index.php/Logos](http://iphonedevwiki.net/index.php/Logos)

Makefile
[http://www.gnu.org/software/make/manual/html_node/Makefiles.html](http://www.gnu.org/software/make/manual/html_node/Makefiles.html)

debian的官网
[http://www.debian.org/doc/debian-policy/ch-controlfields.html](http://www.debian.org/doc/debian-policy/ch-controlfields.html)

debian:
[http://www.debian.org/doc/debian-policy](http://www.debian.org/doc/debian-policy)

Theos安装:
[https://github.com/theos/theos/wiki/Installation](https://github.com/theos/theos/wiki/Installation)

## 2、Theos简介

Theos是一个越狱开发工具包，由iOS越狱界知名人士Dustin Howett 开发并分享到GitHub上。Theos与其他越狱开发工具相比，最大的特点就是简单：下载安装简单、Logos语法简单、编译发布简单，可以让使用者把精力都放在开发工作上去。

值得一提的是，越狱开发中常用的另一工具iOSOpenDev是整合在Xcode里的，熟悉Xcode的朋友可能会对它更感兴趣。但逆向工程接触底层知识较多，很多东西无法自动化，因此推荐使用整合度并不算高的Theos，当你手动完成一个又一个练习时，对逆向工程的理解一定会更深。

**下文与theos相关的内容，最好以官方为准，内容都在不断的更新中。比如安装这块，可以参考https://github.com/theos/theos/wiki/Installation**

**其实很多的问题，与其百度google，不如直接查看Issues，不少问题在里面都有靠谱的答案**

## 3、安装Theos


首先需要的环境和工具如下：

- curl
- git
- make
- openssh
- perl
- rsync
- dpkg (port or homebrew install on Mac OS X)
- python (if on Windows)

没有的就用 brew install **



##### 3.1 安装Xcode与Command Line Tools：

安装Xcode，其中附带了Command Line Tools。

##### 3.2 下载Theos 

[最好参见官方安装文档，这是最新的](https://github.com/theos/theos/wiki/Installation)

安装的路径有:~/theos, /opt/theos, or /var/theos.
安装方式有两种，首先介绍zip安装，目前已经不推荐：

1、方法1：使用下载包安装

从GitHub上下载Theos，操作如下：

	$ export THEOS=/opt/theos
	sudo git clone git://github.com/DHowett/theos.git $THEOS
	Password:
	Cloning into '/opt/theos'...
	...
	
然后设置软连接：

	$ open ~/.bash_profile
	$ export THEOS=/opt/theos
	$ export PATH=$THEOS/bin:$PATH
	$ source ~/.bash_profile

2、方法1：直接使用git 安装：推荐：

注意以下如果不行，就加上sudo：

```
$ brew install dpkg ldid
$ brew install --HEAD hbang/repo/deviceconsole  # (not required, but very useful)
$ cd /opt/theos
$ git clone --recursive https://github.com/theos/theos.git 
$ sudo chown $(id -u):$(id -g) theos 

$ curl https://ghostbin.com/ghost.sh -o $THEOS/bin/ghost
$ chmod +x $THEOS/bin/ghost

```

##### 3.2.1 设置环境变量

```
在 ~/.bash_profile写入： 并执行
export THEOS=/opt/theos
export PATH=$THEOS/bin:$PATH 
export THEOS_DEVICE_IP=Jiang.local THEOS_DEVICE_PORT=22
#手机的IP。如果知道手机IP，就直接将Jiang.local替换成实际的IP即可。

```

##### 3.3  update-theos 
可以晚点执行，先创建一个工程，然后在进行如下操作：

```
进入到一个具体的theos工程里面。先make，然后:
$ sudo make update-theos
如果报错：
Makefile:3: /makefiles/common.mk: No such file or directory
Makefile:13: /tweak.mk: No such file or directory
就修改makefile：
include /opt/theos/makefiles/common.mk 
```

然后：

```
$ make update-theos

【在mac上include /opt/theos/makefiles/common.mk 执行  make update-theos 之后，升级成功。但是pro上出现如下的问题...】
报错：
> Updating Theos…
error: cannot open .git/FETCH_HEAD: Permission denied
 如何解决？？？？？？？？？？？
```

##### 3.4 配置ldid

ldid的作用是模拟给iPhone签名的流程，使得你能够在真实的设备上安装越狱的apps/hacks。签名iOS可执行文件的，用以在越狱iOS中取代Xcode自带的codesign。从[http://joedj.net/ldid](http://joedj.net/ldid)下载ldid，把它放在  “/opt/theos/bin/” 下。然后用以下命令赋予它可执行权限：

	$ sudo chmod 777 /opt/theos/bin/ldid
	

##### 3.5 配置CydiaSubstrate
首先运行Theos的自动化配置脚本，操作如下：

	$sudo /opt/theos/bin/bootstrap.sh substrate
	$sudo: /opt/theos/bin/bootstrap.sh: command not found
	[最新版的 theos 已经没有这个脚本bootstrap.sh，可以跳过这步，直接执行后面的操作就可以了。 ]

##### 3.6  安装\配置dpkg-deb

安装port：
下载地址:https://www.macports.org/
安装之后，新开一个终端窗口：

    $ port version
    Version: 2.3.4
然后安装pkg:

    $sudo port install dpkg

配置：

deb是越狱开发安装包的标准格式，dpkg-deb是一个用于操作deb文件的工具，有了这个工具，Theos才能正确地把工程打包成为deb文件。从[https://raw.githubusercontent.com/DHowett/dm.pl/master/dm.pl](https://raw.githubusercontent.com/DHowett/dm.pl/master/dm.pl)拷贝出dm.pl，将其重命名为dpkg-deb.pl后，放到“/opt/theos/bin/”目录下，然后用以下命令赋予其可执行权限：

	$ sudo chmod 777 /opt/theos/bin/dpkg-deb.pl
	

##### 3.7 指定xcode 

给Xcode命令行工具指定路径.【如果是多个xcode 需要指定】,我指定的是Xcode_7_2_1。
$ sudo xcode-select --switch /Applications/Xcode_7_2_1.app/Contents/Developer/

#####  配置Theos NIC templates
(注意：最新的已经包含这几种模板了，不用重复操作。否则下边的用法里，会出现重复的模板...)

Theos NIC templates内置了5种Theos工程类型的模板，方便创建多样的Theos工程。除此以外，还可以从[https://github.com/DHowett/theos-nic-templates/archive/master.zip](https://github.com/DHowett/theos-nic-templates/archive/master.zip)获取额外的5种模板，下载后将解压得到的5个.tar文件复制到/opt/theos/templates/iphone/ 或 /opt/theos/templates/ios/下即可。

##### 3.8  iOS sdks 安装

关于sdk，简述如此：
Xcode 7.3 removed all private frameworks from the SDK. For now, please download iOS 9.2 SDK from https://sdks.website/, extract to 【Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs 】and set TARGET to use that, or just download Xcode 7.2 again (https://developer.apple.com/downloads/) and use xcode-select to switch to it.

sdk下载地址:https://sdks.website/ 

安装位置：

一种是放在xcode的包里，Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs，然后在~/.theosrc设置 TARGET = iphone::9.3:9.0

另一种是放在theos(/opt/theos/sdks)，也是最推荐的：
You can place the sdk folder in $THEOS/sdks and do:
SDKVERSION = 9.3
SYSROOT = $(THEOS)/sdks/iPhoneOS9.3.sdk
in ~/.theosrc

参考：https://github.com/theos/theos/issues/146#issuecomment-240574611

具体操作8.1为例：

    $mkdir -p $THEOS/sdks 或者：$cd /opt/theos/sdks
    $wget https://sdks.website/dl/iPhoneOS8.1.sdk.tbz2
    $tar jxvf iPhoneOS8.1.sdk.tbz2
如果不行，就直接下载zip，然后解压到/opt/theos/sdks

##### 3.9 支持64位设备的操作

    ln -s $THEOS/makefiles/platform/Darwin-arm.mk $THEOS/makefiles/platform/Darwin-arm64.mk
    ln -s $THEOS/makefiles/targets/Darwin-arm $THEOS/makefiles/targets/Darwin-arm64

##### 3.10 libsubstrate.dylib +substrate.h 配置

下载libsubstrate.dylib
https://github.com/zqmiOSRE/CydiaSubstrateResource
，然后copy到/opt/theos/lib
或者：将https://github.com/theos/lib 这里的下载，放进去。

方法2： 现在采用方法2 
因为Theos 的一个 bug,它无法自动生成一个有效的 libsubstrate.dylib 文件,需要手动操作。
在 Cydia 中搜索安装“Cydia Substrate”,然后用 iFunBox 或 scp 等方式将 iOS 上的“/Library/Frameworks/CydiaSubstrate.framework/ CydiaSubstrate ” copy到 OSX 中,将其重命名为 libsubstrate.dylib 后放到“ /opt/theos/vendor/lib/libsubstrate.dylib”中, 替换掉无效的文件即可。

2、拷贝手机文件 /Library/Frameworks/CydiaSubstrate.framework/Headers/CydiaSubstrate.h 到---> $THEOS/include 重命名为 substrate.h

参考http://iphonedevwiki.net/index.php/Theos/Setup

#####  3.11 iOS headers 

下载iphoneheader到/opt/theos/include：

使用 https://github.com/theos/headers【我是这个】
或者 https://github.com/kennytm/iphone-private-frameworks.git

    $git clone https://github.com/kennytm/iphone-private-frameworks.git
    $mv iphoneheaders/* theos/include/

从OSX library中拷贝IOSurfaceAPI.h到theos/include/IOSurface目录下：

      cp /System/Library/Frameworks/IOSurface.framework/Headers/IOSurfaceAPI.h theos/include/IOSurface 

给IOSurfaceAPI.h打补丁，注释掉IOSurfaceCreateXPCObject 和IOSurfaceLookupFromXPCObject。
注释后的结果是：

```
/* This call lets you get an xpc_object_t that holds a reference to the IOSurface.
   Note: Any live XPC objects created from an IOSurfaceRef implicity increase the IOSurface's global use
   count by one until the object is destroyed. */

/*xpc_object_t IOSurfaceCreateXPCObject(IOSurfaceRef aSurface) XPC_RETURNS_RETAINED
    IOSFC_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_NA);*/


/* This call lets you take an xpc_object_t created via IOSurfaceCreatePort() and recreate an IOSurfaceRef from it. */

/*IOSurfaceRef IOSurfaceLookupFromXPCObject(xpc_object_t xobj) CF_RETURNS_RETAINED
    IOSFC_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_NA);
*/
```


#### 报错汇总

```
报错1
$ make update-theos   会报错 make: *** No rule to make target `update-theos'.  Stop.是因为：  then you are either not currently in a project directory, or are using a version of Theos older than this feature

报错2
Makefile:3: /makefiles/common.mk: No such file or directory
Makefile:13: /tweak.mk: No such file or directory
Makefile修改路径：
include /opt/theos/makefiles/common.mk 

```

###  手机端配置 

参考http://iphonedevwiki.net/index.php/Theos/Setup/iOS

手机端配置
建议开VPN，然后进入Cydia，先添加两个源

http://coolstar.org/publicrepo

http://nix.howett.net/theos

刷新后，搜索安装以下所需软件

BigBoss Recommended Tools
Perl
Theos
iOS Toolchain

###  ssh等配置

    $ open ~/.bash_profile
添加：

使用pp助手，开启SSH通道。手机mac连同一个网络。


清理：make clean 

$ make package 
$ make install

## 4、Theos用法

##### 4.1 创建工程

1）切换工作目录至常用的iOS工程目录，然后启动：


	$cd /Users/Test/CODE
    $/opt/theos/bin/nic.pl ，启动NIC（New Instance Creator）
	或者使用：
    $nic.pl 路径是： THEOS=/opt/theos 路直接启动的话，需要添加到/usr/bin：$ cd ~  $ $THEOS/bin/nic.pl19
    
结果：
```
	NIC 2.0 - New Instance Creator
	------------------------------
	[1.] iphone/activator_event
  	[2.] iphone/application_modern
  	[3.] iphone/cydget
  	[4.] iphone/flipswitch_switch
  	[5.] iphone/framework
  	[6.] iphone/ios7_notification_center_widget
  	[7.] iphone/library
  	[8.] iphone/notification_center_widget
  	[9.] iphone/preference_bundle_modern
  	[10.] iphone/tool
  	[11.] iphone/tweak
  	[12.] iphone/xpc_service
    Choose a Template (required):
```
可以看到，这里共有多种模板可供选择，其中一些事是Theos的自带模板，一些是之前下载的。在逆向工程初级阶段，所开发程序的主要类型是tweak。

2）选择“19”，即创建一个tweak工程：

	 Choose a Template (required): 11

3）输入tweak的工程名称： 

	Project Name (required): projectBy3cod

4）输入deb包的名字（类似于bundle identifier）： 

	Package Name [com.yourcompany.projectBy3cod]: com.jxc.projectBy3cod

5）输入tweak作者的名字，命令如下： 
	
	Author/Maintainer Name [gyjrong]: 3code

6）输入“MobileSubstrate Bundle filter”，也就是tweak作用对象的bundle identifier：
filter这一项表示要hook的程序，默认是com.apple.springboard，就是hook Spring Board，如果你想hook别的App，这里改成那个App的BundleID.


	...[com.apple.springboard]: com.apple.springboard

7）输入tweak安装完成后需要重启的应用，以进程名表示，如下：

	...[SpringBoard]: SpringBoard 
Done.

简单的7步完成之后，一个名为projectBy3cod的文件夹就在当前目录生成了，该文件夹里就是刚创建的tweak工程。

##### 4.2 定制工程文件
用Theos创建tweak工程非常方便，但简洁的工程框架下目前还是些粗糙的内容，需要进一步加工相关的文件。先来看看刚刚生成的工程目录，如下：

	$ cd projectBy3cod
	$ ls -l
	total 32
	-rw-r--r--  1 gyjrong  admin    57 10 31 18:46 projectBy3cod.plist
	-rw-r--r--  1 gyjrong  admin   274 10 31 18:46 Makefile
	-rw-r--r--  1 gyjrong  admin  1045 10 31 18:46 Tweak.xm
	-rw-r--r--  1 gyjrong  admin   207 10 31 18:46 control

具体的操作参见文章

[越狱开发系列3_Theos_file简介及Logos基本语法](http://3code.info/2016/11/01/2_Theos_ install&run/)


**小技巧： make messages=yes 打印全部的logs信息**


##### 4.3 编译

在完成了Theos的安装后，使用NIC创建了第一个tweak工程，那么现在就剩下最后一步——编译了。完成这一步，一个tweak就算正式完成——我们可以把tweak安装到设备上，开始周而复始的“safe mode”之旅。

$ make clean && make do #啥作用？？

1）编译

编译报错处理:
```
1、$ make
xcrun: error: SDK "iphoneos" cannot be located
==> Error: You do not have an SDK in /Library/Developer/CommandLineTools/Platforms/iPhoneOS.platform/Developer/SDKs.
原因，多个xcode导致路径问题。解决方法：给Xcode命令行工具指定路径.【如果是多个xcode 需要指定】
$ sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/

2、$ make
PluginLoading: Required plug-in compatibility UUID (。。。)for plug-in at path '~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/VVDocumenter-Xcode.xcplugin' not present in DVTPlugInCompatibilityUUIDs

解决方法，回头按照上边的方法： update-theos 成功操作之后，
再次make：
发现中文的",然后重新辨析xm，修改掉，
再次make：提示clang: warning: libstdc++ is deprecated; move to libc++ with a minimum deployment target of iOS 7
Undefined symbols for architecture armv7:....

先完成了上边你的全部设置。编译报错。
3、$ make
clang: warning: libstdc++ is deprecated; move to libc++ with a minimum deployment target of iOS 7
Undefined symbols for architecture armv7:
  "_main", referenced from:
      start in crt1.3.1.o
ld: symbol(s) not found for architecture armv7
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[3]: *** [/Users/Test/CODE/changelockwin/.theos/obj/debug/armv7/changeLockWin.dylib] Error 1。。。。


4、package的原因：【dpkg-deb: error: obsolete compression type 'lzma'; use xz instead】

问题参见：
https://github.com/theos/theos/issues/211 以及
https://github.com/theos/theos/issues/180

因为最新的dpkg已经舍弃了部分旧的方法，而theos还在使用。所以简单粗暴的就是使用低版本的dpkg
地址：[https://bintray.com/homebrew/bottles/dpkg#files.](https://bintray.com/homebrew/bottles/dpkg#files)

首先：brew list --versions
看看自己的版本，需要安装1.18.10甚至更低的版本。
解压到：  /usr/local/Cellar/dpkg/1.18.10 
然后：
$ for i in $(brew --cellar)/dpkg/1.18.10/bin/*; do sed -i s/'@@HOMEBREW_CELLAR@@'/$(brew --cellar)/g "$i"; done
$ brew switch dpkg 1.18.10#选择版本
$ brew pin dpkg #禁止升级

如果还有这个警告，可以忽略之。

```

```




Theos采用“make”命令来编译Theos工程。在Theos工程目录下运行make命令，如下：

	$ make
	Making all for tweak iOSREProject...
	Preprocessing Tweak.xm...
	Compiling Tweak.xm...
	Linking tweak ...
	Stripping ...
	Signing ...
	
从输出的信息看，Theos完成了预处理、编译、签名等一系列动作，此时会发现当前目录下多了一个新的“obj”文件夹，如下：

	$ ls -l
	Makefile Tweak.xm control iOSREProject.plist  obj  /opt/theos
	
里面有一个.dylib文件，如下：

	$ ls -l ./obj
	Tweak.xm.b1748661.o	iOSREProject.dylib
	
它就是tweak的核心。

##### 4.4 打包

打包使用的“make package”命令来自于Theos本身，其实就是先执行“make”命令，然后再执行“dpkg-deb”命令，如下：

```
$ make package
Making all for tweak iOSREProject...
 Preprocessing Tweak.xm...
 Compiling Tweak.xm...
 Linking tweak iOSREProject...
 Stripping iOSREProject...
 Signing iOSREProject...
Making stage for tweak iOSREProject...
dm.pl: building package `com.iosre.iosreproject' in `./com.iosre.iosreproject_0.0.1-7_iphoneos-arm.deb'.
```
上面生成了一个名为“com.iosre.iosreproject_0.0.1-7_iphoneos-arm.deb”的文件，这就是可以最终发布的安装包。

“make package”命令还有一个很重要的功能。在执行完“make package”之后，除了“obj”文件夹外，你会发现tweak工程目录下还生成了一个“_”文件夹，如下：

```
$ ls -l
total 40
-rw-r--r--  1 snakeninny  staff   262 Dec  3 09:20 Makefile
-rw-r--r--  1 snakeninny  staff     0 Dec  3 11:28 Tweak.xm
drwxr-xr-x  4 snakeninny  staff   136 Dec  3 11:35 _
-rw-r--r--  1 snakeninny  staff  2396 Dec  3 11:35 com.iosre.iosreproject_0.0.1-7 _iphoneos-arm.deb
-rw-r--r--  1 snakeninny  staff   223 Dec  3 09:05 control
-rw-r--r--@ 1 snakeninny  staff   175 Dec  3 09:48 iOSREProject.plist
drwxr-xr-x  5 snakeninny  staff   170 Dec  3 11:35 obj
lrwxr-xr-x  1 snakeninny  staff    11 Dec  3 09:05 theos -> /opt/theos
```

这个文件夹是干什么的？打开它，可以看到2个文件夹，分别是“DEBIAN”和“Library”：

```
$ ls -l _
total 0
drwxr-xr-x  3 snakeninny  staff  102 Dec  3 11:35 DEBIAN
drwxr-xr-x  3 snakeninny  staff  102 Dec  3 11:35 Library
其中“DEBIAN”里只有tweak工程里的control文件，Theos在编译[…]”
```
其中“DEBIAN”里只有tweak工程里的control文件，Theos在编译过程中向control文件里稍稍增加了几个字段而已，如下：

```
snakeninnysiMac:iosreproject snakeninny$ ls -l _/DEBIAN
total 8
-rw-r--r--  1 snakeninny  staff  245 Dec  3 11:35 control
```

“Library”的目录结构如下：
![](https://ooo.0o0.ooo/2016/11/01/58184149d2d6e.jpeg)


对比生成deb的包内容：

```
$ dpkg -c com.iosre.iosreproject_0.0.1-7_iphoneos-arm.deb
drwxr-xr-x snakeninny/staff  0 2014-12-03 11:35 ./
drwxr-xr-x snakeninny/staff  0 2014-12-03 11:35 ./Library/
drwxr-xr-x snakeninny/staff  0 2014-12-03 11:35 ./Library/MobileSubstrate/
drwxr-xr-x snakeninny/staff  0 2014-12-03 11:35 ./Library/MobileSubstrate/DynamicLibraries/
-rwxr-xr-x snakeninny/staff 98784 2014-12-03 11:35 ./Library/MobileSubstrate/DynamicLibraries/iOSREProject.dylib
-rw-r--r-- snakeninny/staff   175 2014-12-03 11:35 ./Library/MobileSubstrate/DynamicLibraries/iOSREProject.plist
```

以及在Cydia中iOSREProject的文件系统，如下

![](https://ooo.0o0.ooo/2016/11/01/581841f406d47.png)

可以看到，三者是完全相同的。到这里，你可能也猜到了，这个deb包其实就是由“DEBIAN”提供debian信息，“Library”提供实际文件的简单组合。事实上，还可以在工程目录下创建一个名为“layout”的文件夹，然后把工程打包成deb并安装到iOS中，此时“layout”中的所有文件会被解包到iOS文件系统的相同位置（这里的“layout”相当于iOS中的根目录“/”），这极大扩充了deb包的作用范围。下面用一个小示例佐以说明。

回到刚才的iOSREProject中，在Terminal中输入“make clean”及“rm*.deb”，将工程恢复到最初的状态，如下：

```
$ make clean
rm -rf ./obj
rm -rf "/Users/snakeninny/Code/iosreproject/_"
$ rm *.deb
$ ls -l
total 32
-rw-r--r--  1 snakeninny  staff  262 Dec  3 09:20 Makefile
-rw-r--r--  1 snakeninny  staff    0 Dec  3 11:28 Tweak.xm
-rw-r--r--  1 snakeninny  staff  223 Dec  3 09:05 control
-rw-r--r--@ 1 snakeninny  staff  175 Dec  3 09:48 iOSREProject.plist
lrwxr-xr-x  1 snakeninny  staff   11 Dec  3 09:05 theos -> /opt/theos
```

然后生成一个空的“layout”目录，如下：

	$ mkdir layout
并在“layout”下随便放一些空文件，如下：

```
$ touch ./layout/1.test
$ mkdir ./layout/Developer
$ touch ./layout/Developer/2.test
$ mkdir -p ./layout/var/mobile/Library/Preferences
$ touch ./layout/var/mobile/Library/Preferences/3.test
```
最后用“make package”打包，并将生成的deb文件拷贝到iOS中，用iFile安装。然后在Cydia中查看iOSREProject的文件系统，如图下

![](https://ooo.0o0.ooo/2016/11/01/581842e2adbd5.jpeg)
 
除“DEBIAN”以外的所有文件都被解包到了iOS文件系统的相同位置，本来不存在的中间文件夹也被自动创建。deb包的玄机还有很多，这里也只是管中窥豹，更全面的介绍请移步[http://www.debian.org/doc/debian-policy](http://www.debian.org/doc/debian-policy)，官方文档总是最好的学习资料。



##### 4.5 安装

最后，要把这个deb文件安装到iOS中去。安装的方法多种多样，这里介绍两种最具代表性的：图形界面安装法和命令行安装法。

- 图形界面安装法

这个方法确实简单：通过iFunBox等软件把deb拖到iOS里去，然后用iFile安装它，最后重启iOS。虽然全过程都由图形界面操作，但人机交互太多，又要动电脑又要滑手机，一来二去非常繁琐，并不适用于tweak开发。

- 命令行安装法

这个方法要用到简单的ssh命令，故而要求越狱的iOS安装了OpenSSH，


打开ssh通道，这里使用PP助手实现，之后按照其操作：

$ ssh root@localhost -p 2222

输入密码即可。


下面具体介绍安装法。

首先，需要在Makefile的最上一行加上本机IP地址，如下：

```
THEOS_DEVICE_IP = iOSIP
ARCHS = armv7 arm64
TARGET = iphone:latest:8.0
```

然后调用“make package install”命令完成编译打包安装一条龙服务，如下：

```
$ make package install
Making all for tweak iOSREProject...
 Preprocessing Tweak.xm...
 Compiling Tweak.xm...
 Linking tweak iOSREProject...
 Stripping iOSREProject...
 Signing iOSREProject...
Making stage for tweak iOSREProject...
dm.pl: building package `com.iosre.iosreproject:iphoneos-arm' in `./com.iosre.iosreproject_0.0.1-15_iphoneos-arm.deb'
install.exec "cat > /tmp/_theos_install.deb; dpkg -i /tmp/_theos_install.deb && rm /tmp/_theos_install.deb" < "./com.iosre.iosreproject_0.0.1-15_iphoneos-arm.deb"
root@iOSIP's password: 
Selecting previously deselected package com.iosre.iosreproject.
(Reading database ... 2864 files and directories currently installed.)
Unpacking com.iosre.iosreproject (from /tmp/_theos_install.deb) ...
Setting up com.iosre.iosreproject (0.0.1-15) ...
install.exec "killall -9 SpringBoard"
root@iOSIP's password:
```

从以上信息可以看到，Theos在整个安装过程中要求我们输入两次root密码。虽然多次输入密码给人很安全的感觉，但实在是太麻烦了。好在通过设置iOS的authorized_keys可以省略SSH输密码的步骤，让“make package install”真正地从“一只多脚虫”变成“一条飞天龙”，具体步骤如下：

1）删除“/Users/snakeninny/.ssh/known_hosts”中iOSIP对应的条目。
假设iOS的IP地址是iOSIP。编辑“/Users/snakeninny/.ssh/known_hosts”，找到iOSIP所在的那一行，如下：

```
iOSIP ssh-rsa hXFscxBCVXgqXhwm4PUoUVBFWRrNeG6gVI3Ewm4dqwusoRcyCxZtm5bRiv4bXfkPjsRkWVVfrW3uT52Hhx4RqIuCOxtWE7tZqc1vVap4HIzUu3mwBuxog7WiFbsbbaJY4AagNZmX83Wmvf8li5aYMsuKeNagdJHzJNtjM3vtuskK4jKzBkNuj0M89TrV4iEmKtI4VEoEmHMYzWwMzExXbyX5NyEg5CRFmA46XeYCbcaY0L90GExXsWMMLA27tA1Vt1ndHrKNxZttgAw31J90UDnOGlMbWW4M7FEqRWQsWXxfGPk0W7AlA54vaDXllI5CD5nLAu4VkRjPIUBrdH5O1fqQ3qGkPayhsym3g0VZeYgU4JAMeFc3
```
完整删掉这一行。
2）生成authorized_keys。

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/snakeninny/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/snakeninny/.ssh/id_rsa.
Your public key has been saved in /Users/snakeninny/.ssh/id_rsa.pub.
……
$ cp /Users/snakeninny/.ssh/id_rsa.pub ~/authorized_keys
```
就会在用户目录下生成authorized_keys。
3）配置iOS。

```
FunMaker-5:~ root# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /var/root/.ssh/id_rsa.
Your public key has been saved in /var/root/.ssh/id_rsa.pub.
……
FunMaker-5:~ root# logout
Connection to iOSIP closed.
$ scp ~/authorized_keys root@iOSIP:/var/root/.ssh
The authenticity of host 'iOSIP (iOSIP)' can't be established.
RSA key fingerprint is 75:98:9a:05:a3:27:2d:23:08:d3:ee:f4:d1:28:ba:1a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'iOSIP' (RSA) to the list of known hosts.
root@iOSIP's password: 
authorized_keys                            100%  408    0.4KB/s   00:00
```

重新使用ssh命令进入iOS试试看，还需要输密码吗？此时，“make package install”真正变成了一次配置，一键安装，一劳永逸！
（4）清理

Theos提供了方便的工程清理命令“make clean”，其“实际作用就是依次执行“rm-rf./obj”和“rm-rf"/Users/snakeninny/Code/iosre/_"”两个命令，从而删除“make”和“make package”命令生成的文件夹。也可以用“rm*.deb”，删除“make package”命令生成的所有deb文件。

## 5　Theos开发tweak示例

接下来将以一个最简单的tweak为例来进行讲解。安装了该程序之后，每次重启SpringBoard都将会弹出一个UIAlertView。

##### 5.1 用Theos新建tweak工程“iOSREGreetings”

新建iOSREGreetings工程的命令如下：

```
$ /opt/theos/bin/nic.pl
NIC 2.0 - New Instance Creator
------------------------------
  [1.] iphone/application
  [2.] iphone/cydget
  [3.] iphone/framework
  [4.] iphone/library
  [5.] iphone/notification_center_widget
  [6.] iphone/preference_bundle
  [7.] iphone/sbsettingstoggle
  [8.] iphone/tool
  [9.] iphone/tweak
  [10.] iphone/xpc_service
Choose a Template (required): 9
Project Name (required): iOSREGreetings
Package Name [com.yourcompany.iosregreetings]: com.iosre.iosregreetings
Author/Maintainer Name [snakeninny]: snakeninny
[iphone/tweak] MobileSubstrate Bundle filter [com.apple.springboard]: com.apple.springboard
[iphone/tweak] List of applications to terminate upon installation (space-separated, '-' for none) [SpringBoard]: 
Instantiating iphone/tweak in iosregreetings/...
Done.
```

##### 5.2.编辑Tweak.xm

编辑后的Tweak.xm内容如下：

```
%hook SpringBoard
- (void)applicationDidFinishLaunching:(id)application
{
    %orig;
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Come to http://bbs.iosre.com for more fun!" message:nil delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
    [alert show];
    [alert release];
}
%end
```

##### 5.3.编辑Makefile及control

编辑后的Makefile内容如下：

```
THEOS_DEVICE_IP = iOSIP
ARCHS = armv7 arm64
TARGET = iphone:latest:8.0
include theos/makefiles/common.mk
TWEAK_NAME = iOSREGreetings
iOSREGreetings_FILES = Tweak.xm
iOSREGreetings_FRAMEWORKS = UIKit
include $(THEOS_MAKE_PATH)/tweak.mk
after-install::
      install.exec "killall -9 SpringBoard"
```

编辑后的control内容如下：

```
Package: com.iosre.iosregreetings
Name: iOSREGreetings
Depends: mobilesubstrate, firmware (>= 8.0)
Version: 1.0
Architecture: iphoneos-arm
Description: Greetings from iOSRE!
Maintainer: snakeninny
Author: snakeninny
Section: Tweaks
Homepage: http://3code.info
```
以上代码非常简单，当SpringBoard的applicationDidFinishLaunching:函数得到调用时，代表SpringBoard的启动过程已经结束。钩住（hook）这个函数，调用%orig完成它的原始操作，然后弹出一个自定义的UIAlertView；这样一来，每次重启SpringBoard都会弹“出一个对话框。你看懂了吗？

准备就绪，在Terminal中敲入“make package install”，
如果出现错误:``` PluginLoading: Required plug-in compatibility UUID 8A66E736-A720-4B3C-92F1-33D9962C69DF for plug-in at path '~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/VVDocumenter-Xcode.xcplugin' not present in DVTPlugInCompatibilityUUIDs
==> Error: The vendor/include and/or vendor/lib directories are missing. Please run `git submodule update --init --recursive` in your Theos directory. More information: https://github.com/theos/theos/wiki/Installation. ```

就在 Theos 目录里运行 $git submodule update --init --recursive

待SpringBoard重启之后会看到结果，简单粗暴。是的，仅仅是这样一些小小的改动，就已经可以改变App的行为了。此时，封闭的iOS已经向我们打开了大门……

因为有Theos这样的开发工具存在，修改闭源的iOS程序变得前所未有的方便。不过在前面也提到了，现在的App工程量越来越大，class-dump头文件也越来越多，要从浩如烟海的函数名中筛选出我们感兴趣的目标，比确定目标后编写代码还要难得多。面对成千上万行代码，如果没有其他工具辅助分析，逆向工程简直是一场噩梦，让人一筹莫展。那么接下来，就轮到这些辅助分析工具隆重登场了。
 



 
 




 



