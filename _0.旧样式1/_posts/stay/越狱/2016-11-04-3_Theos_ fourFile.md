---
layout: post
title:  "iOS逆向系列3_Theos_file简介及Logos基本语法"
categories: Reverse
tags:  Theos  Logos
author: 3行代码
---

* content
{:toc}

**待整理**

在iosOpenDev之前，很多ios插件都使用theos编译开发，现在使用theos开发的人也不在少数。theos 有自己的模板用于开发一系列的插件程序，所以在早期开发的插件中基本上都是使用theos。
怎样安装theos，在前边的文章已有介绍：[传送门]()

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

## 2、Makefile

##### 2.1 基本配置

 Makefile文件指定工程用到的文件、框架、库等信息。projectBy3code的Makefile内容以及解释如下：
 
	#固定写法:系统变量，不要更改。
    #告诉theOS在编译脚本中包括共同的make命令，避免做重复的make编译工作
	include $(THEOS)/makefiles/common.mk
	
	#tweak的名字，即用Theos创建工程时指定的“Project Name”，跟control文件中的“Name”字段对应，不要更改。
	TWEAK_NAME = projectBy3code

	#tweak包含的源文件（不包括头文件），多个文件间以空格分隔，如： projectBy3code_FILES = Tweak.xm Hook.xm New.x ObjC.m ObjC++.mm.头文件是可以按需修改的。
    #这里是需要编译的文件列表，注意：不要把头文件添加到这里。如果你要添加一个新的.m或者.mm文件到项目中，确保在这里添加新的文件的名称，否则将不会建立编译连接。
	projectBy3code_FILES = Tweak.xm

	#根据不同的Theos工程类型，通过include命令指定不同的.mk文件；在逆向工程初级阶段，我们开发的一般是Application、Tweak和Tool三种类型的程序，它们对应的.mk文件分别是application.mk、tweak.mk和tool.mk，可以按需更改。	
	include $(THEOS_MAKE_PATH)/tweak.mk
	
	#tweak安装之后杀掉SpringBoard进程，好让CydiaSubstrate在进程启动时加载对应的dylib。
	after-install::
			install.exec "killall -9 SpringBoard; killall -9 Instantiating; killall -9 iphone/tweak; killall -9 in; killall -9 projectBy3code/..."
	
 
是不是非常简单？Makefile里的默认内容确实非常简单，如何指定SDK版本？怎么导入framework？lib文件在哪里链接？，面包会有的，牛奶也会有的。下面介绍：

##### 2.2 指定处理器架构

**这里不需要明确的指出，可以不加这一条！！原因是5s使用的64位处理器，不能用armv7指令集，所以需要在Makefile里将armv7更改为arm64，因此不要加，可能得不偿失.除非明确，否则不要加如下代码**

	ARCHS = armv7 arm64

上面的语句在表示不同的处理器架构，采用arm64架构的App不兼容armv7/armv7s架构，必须适配arm64架构的dylib。
You shouldn't need to explicitly set ARCHS unless you have a very good reason. Theos will always ensure it is set to an appropriate value for your SDK and deployment targets.


##### 2.3 指定SDK版本

默认是：iphone:clang:latest:latest。意即meaning to build for iOS, using clang, with the latest SDK version, deploying to support iOS versions matching the SDK version, or newer。

SDK可以放在xcode的包路径里面，或者最好的是放在theos里，然后设置路径即可。
方法如下：
 you should edit ~/.theosrc (which probably doesn’t exist, so create it) and insert something along the lines of:
 
SDKVERSION = 9.3
SYSROOT = $(THEOS)/sdks/iPhoneOS9.3.sdk

TARGET = iphone::9.2:9.0

Which indicates that you are building for iOS, using SDK 9.2, and intending to support iOS 9.0 and newer.

	TARGET = iphone:latest:8.0

 

比如：TARGET = iphone:8.1:8.0
   上面的语句即指定采用8.1版本的SDK，且发布对象为iOS 8.0及以上版本。也可以把“Base SDK”设置为“latest”，指定以Xcode附带的最新版本SDK编译，如：TARGET = iphone:latest:8.0
   
##### 2.4 导入framework
这里包括你想用到框架的名称

	projectBy3code_FRAMEWORKS =  UIKit CoreTelephony CoreAudio
	
例如：projectBy3code_FRAMEWORKS = UIKit CoreTelephony CoreAudio
上面的语句所展示的功能没什么多说的，但既然是tweak开发，很多朋友关注的应该是如何导入private framework吧？很简单，用下面的语句即可：

	projectBy3code_PRIVATE_FRAMEWORKS = AppSupport ChatKit IMCore
例如：projectBy3code_PRIVATE_FRAMEWORKS = AppSupport ChatKit IMCore

虽然只是多了个“PRIVATE”，但有一点要注意：private framework是AppStore开发所不允许使用的，它的内容在每个iOS版本之间可能发生变化，在导入之前，一定要确定导入的private framework确实存在。举一个例子，如果你的tweak打算兼容iOS 7和iOS 8两个版本，那么Makefile可写成如下内容：

```
ARCHS = armv7 arm64
TARGET = iphone:latest:7.0
include theos/makefiles/common.mk
TWEAK_NAME = projectBy3code
projectBy3code_FILES = Tweak.xm
projectBy3code_PRIVATE_FRAMEWORK = BaseBoard
include $(THEOS_MAKE_PATH)/tweak.mk
after-install::
      install.exec "killall -9 SpringBoard"
```  
上面的语句可以成功编译和链接，并不会报错。但是，因为BaseBoard这个private framework只存在于8.0及以上版本的SDK里，在iOS 7里是没有的，所以这个tweak在iOS 7中会因找不到framework而无法正常工作。这种情况可以通过弱链接（谷歌搜索“makefile weak linking”）或dlopen()、dlsym()和dlclose()系列函数动态调用private framework来解决。

##### 2.5 链接Mach-O对象（Mach-O object）

	projectBy3code_LDFLAGS = -lx
	
Theos采用GNU Linker来链接Mach-O对象，包括.dylib、.a和.o。在Terminal中输入“man ld”，定位到“-lx”部分，它是这么写的：

	“-lx This option tells the linker to search for libx.dylib or libx.a in the library search path.If string x is of the form y.o,then that file is searched for in the “same places,but without prepending`lib’or appending`.a’or`.dylib’to the filename.”


大致意思是说，-lx代表链接libx.a或libx.dylib，即给“x”加上“lib”的前缀，以及“.a”或“.dylib”的后缀；如果x是“y.o”的形式，则直接链接y.o，不加任何前缀或后缀。iOS支持链接的Mach-O对象全是以“libx.dylib”和“y.o”形式命名的，完全兼容GNU Linker。“这样，链接Mach-O对象就很方便了。例如，要链接libsqlite3.0.dylib、libz.dylib和dylib1.o，像下面这么写就可以了：

	projectBy3code_LDFLAGS = -lz –lsqlite3.0 –dylib1.o
		
稍后还有一个字段需要介绍，但一般来说，Makefile中定义了以上字段就已经完全够用了；更详细的Makefile介绍，可以参阅[http://www.gnu.org/software/make/manual/html_node/Makefiles.html](http://www.gnu.org/software/make/manual/html_node/Makefiles.html)



## 3、Tweak.xm
 用Theos创建tweak工程，默认生成的源文件是Tweak.xm。“xm”中的“x”代表这个文件支持Logos语法，如果后缀名是单独一个“x”，说明源文件支持Logos和C语法；如果后缀名是“xm”，说明源文件支持Logos和C/C++语法，与“m”和“mm”的区别类似。Tweak.xm的内容如下：

```
/* How to Hook with Logos
Hooks are written with syntax similar to that of an Objective-C @implementation.”
“You don't need to #include <substrate.h>, it will be done automatically, as will
the generation of a class list and an automatic constructor.
%hook ClassName
// Hooking a class method
+ (id)sharedInstance {
        return %orig;
}
// Hooking an instance method with an argument.
- (void)messageName:(int)argument {
      %log; // Write a message about this call, including its class, name and arguments, to the system log.
      %orig; // Call through to the original function with its original arguments.
      %orig(nil); // Call through to the original function with a custom argument.
      // If you use %orig(), you MUST supply all arguments (except for self and _cmd, the automatically generated ones.)
}
// Hooking an instance method with no arguments.
- (id)noArguments {
      %log;
      id awesome = %orig;
      [awesome doSomethingElse];
      return awesome;
}
// Always make sure you clean up after yourself; Not doing so could have grave consequences!
%end
*/
```

这就是最基本的Logos语法，包含%hook、%log、%orig这3个预处理指令，它们的作用如下。

##### 3.1 %hook

指定需要hook的class，必须以%end结尾，如下：

	%hook SpringBoard
	- (void)_menuButtonDown:(id)down
	{
        NSLog(@"You've pressed home button.");
        %orig; // call the original _menuButtonDown:
	}
	%end
	
这段代码的意思是钩住（hook）SpringBoard类里的_menuButtonDown:函数，先将一句话写入syslog，再执行函数的原始操作。

##### 3.2 %log

该指令在%hook内部使用，将函数的类名、参数等信息写入syslog，可以以%log([(<type>)<expr>,...])的格式追加其他打印信息，如下：

	%hook SpringBoard
	- (void)_menuButtonDown:(id)down
	{
      %log((NSString *)@"iOSRE", (NSString *)@"Debug");
      %orig; // call the original _menuButtonDown:
	}
	%end
打印结果如下：

	Dec  3 10:57:44 FunMaker-5 SpringBoard[786]: -[<SpringBoard: 0x150eb800> _menuBu-ttonDown:+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
        Timestamp:           75607608282
        Total Latency:       20266 us
        SenderID:            0x0000000100000190
        BuiltIn:             1
        AttributeDataLength: 16
        AttributeData:       01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
        ValueType:           Absolute
        EventType:           Keyboard
        UsagePage:           12
        Usage:               64
        Down:                1
        +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
        ]: iOSRE, Debug


##### 3.3 %orig

该指令在%hook内部使用，执行被钩住（hook）的函数的原始代码，如下：

	%hook SpringBoard
	- (void)_menuButtonDown:(id)down
	{
      NSLog(@"You've pressed home button.");
      %orig; // call the original _menuButtonDown:
	}
	%end

如果去掉%orig，那么原始函数不会得到执行，例如：

	%hook SpringBoard
	- (void)_menuButtonDown:(id)down
	{
      NSLog(@"You've pressed home button but it's not functioning.");
	}
	%end

还可以利用%orig更改原始函数的参数，例如：

	%hook SBLockScreenDateViewController
	- (void)setCustomSubtitleText:(id)arg1 withColor:(id)arg2
	{
      %orig(@"iOS 8 App Reverse Engineering", arg2);
	}
	%end
上边就修改掉了锁屏显示，原本显示日期的地方就变掉了。
 

***除了%hook、%log、%orig以外，Logos常用的预处理指令还有%group、%init、%ctor、%new、%c，下面继续逐一介绍。***

##### 3.4 %group

该指令用于将%hook分组，便于代码管理及按条件初始化分组（含义稍后有详细解释），必须以%end结尾；一个%group可以包含多个%hook，所有不属于某个自定义group的%hook会被隐式归类到%group_ungrouped中。%group的用法如下：

	%group iOS7Hook
	%hook iOS7Class
	- (id)iOS7Method
	{
        id result = %orig;
        NSLog(@"This class & method only exist in iOS 7.");
        return result;
	}
	%end
	%end // iOS7Hook
	%group iOS8Hook
	%hook iOS8Class
	- (id)iOS8Method
	{
        id result = %orig;
        NSLog(@"This class & method only exist in iOS 8.");
        return result;
	}
	%end
	%end // iOS8Hook
	%hook SpringBoard
	-(void)powerDown
	{
        %orig;
	}
	%end
	
这段代码的意思是在%group iOS7Hook中钩住iOS7Class的iOS7Method，在%group iOS8Hook中钩住iOS8Class的iOS8Method函数，然后在%group_ungrouped中钩住SpringBoard类的powerDown函数。

**需要注意的是，%group必须配合下面的%init使用才能生效。**

##### 3.5 %init

该指令用于初始化某个%group，必须在%hook或%ctor内调用；如果带参数，则初始化指定的group，如果不带参数，则初始化_ungrouped，如下：

	#ifndef kCFCoreFoundationVersionNumber_iOS_8_0
	#define kCFCoreFoundationVersionNumber_iOS_8_0 1140.10
	#endif
	%hook SpringBoard
	- (void)applicationDidFinishLaunching:(id)application
	{
        %orig;
        %init; // Equals to %init(_ungrouped)
        if (kCFCoreFoundationVersionNumber >= kCFCoreFoundationVersionNumber_iOS_7_0 && kCFCoreFoundationVersionNumber < kCFCoreFoundationVersionNumber_iOS_8_0) %init(iOS7Hook);
        if (kCFCoreFoundationVersionNumber >= kCFCoreFoundationVersionNumber_iOS_8_0) init(iOS8Hook);
	}
	%end
	
**只有调用了%init，对应的%group才能起作用，切记切记！**

##### 3.6 %ctor

tweak的constructor，完成初始化工作；如果不显式定义，Theos会自动生成一个%ctor，并在其中调用%init(_ungrouped)。因此，

	%hook SpringBoard
	- (void)reboot
	{
        NSLog(@"If rebooting doesn't work then I'm screwed.");
        %orig;
	}
	%end”
	
可以成功生效，因为Theos隐式定义了如下内容：

	%ctor
	{
        %init(_ungrouped);
	}
	
而

	%hook SpringBoard
	- (void)reboot
	{
        NSLog(@"If rebooting doesn't work then I'm screwed.");
        %orig;
	}
	%end
	%ctor
	{
        // Need to call %init explicitly!
	}
	
里的%hook无法生效，因为这里显式定义了%ctor，却没有显式调用%init，%group(_ungrouped)不起作用。%ctor一般可以用来初始化%group，以及进行MSHookFunction等操作，如下：

	#ifndef kCFCoreFoundationVersionNumber_iOS_8_0
	#define kCFCoreFoundationVersionNumber_iOS_8_0 1140.10
	#endif
	%ctor
	{
        %init;
        if (kCFCoreFoundationVersionNumber >= kCFCoreFoundationVersionNumber_iOS_7_0 && kCFCoreFoundationVersionNumber < kCFCoreFoundationVersionNumber_iOS_8_0) %init(iOS7Hook);
        if (kCFCoreFoundationVersionNumber >= kCFCoreFoundationVersionNumber_iOS_8_0) %init(iOS8Hook);
        MSHookFunction((void *)&AudioServicesPlaySystemSound,
	                (void *)&replaced_AudioServicesPlaySystemSound,
                (void **)&original_AudioServicesPlaySystemSound);
	}
	
注意，%ctor不需要以%end结尾。

##### 3.7 %new

在%hook内部使用，给一个现有class添加新函数，功能与class_addMethod相同。它的用法如下：

	%hook SpringBoard
	%new
	- (void)namespaceNewMethod
	{
      NSLog(@"We've added a new method to SpringBoard.");
	}
	%end
	
有的朋友可能会问，Objective-C的category语法也可以给现有class添加新函数，为什么还需要%new呢？其实原因就在于category与class_addMethod的区别，前者是静态的，而后者是动态的。那么在这种情况下，静态还是动态，有什么关系呢？当然有关系，尤其是当class来自某个可执行文件的时候。举个例子，上面的代码给SpringBoard类添加了一个新方法，如果使用category，代码应该是下面这样：

	@interface SpringBoard (iOSRE)
	- (void)namespaceNewMethod;
	@end
	@implementation SpringBoard (iOSRE)
	- (void)namespaceNewMethod
	{
      NSLog(@"We've added a new method to SpringBoard.");
	}
	@end
	
如果尝试编译上面的代码，会得到“error：cannot find interface declaration for‘SpringBoard’”的报错信息，即编译器找不到SpringBoard类的定义。可以构造一个SpringBoard的定义，骗过编译器，如下：

	@interface SpringBoard : NSObject
	@end
	@interface SpringBoard (iOSRE)
	- (void)namespaceNewMethod;
	@end
	@implementation SpringBoard (iOSRE)
	- (void)namespaceNewMethod
	{
      NSLog(@"We've added a new method to SpringBoard.");
	}
	@end
	
重新编译，仍然会报错，如下：

	Undefined symbols for architecture armv7:
	"_OBJC_CLASS_$_SpringBoard", referenced from:
      l_OBJC_$_CATEGORY_SpringBoard_$_iOSRE in Tweak.xm.b1748661.o
	ld: symbol(s) not found for architecture armv7
	clang: error: linker command failed with exit code 1 (use -v to see invocation)

ld找不到“SpringBoard”的定义。一般来说，iOS程序员在碰到这个错误时的第一反应是：“是不是忘了导入哪个framework？”，但是转念一想，SpringBoard类是SpringBoard这个App里的一个类，而不是一个framework，要怎么导入？现在你是不是觉得%new非常可爱了呢？

##### 3.8 %c

该指令的作用等同于objc_getClass或NSClassFromString，即动态获取一个类的定义，在%hook或%ctor内使用。
Logos的预处理指令还有%subclass和%config，感兴趣的读者可以移步http://iphonedevwiki.net/index.php/Logos一探究竟


## 4、control

control文件记录了deb包管理系统所需的基本信息，会被打包进deb包里。projectBy3code里control文件的内容如下：

	Package: com.jxc.projectBy3code
	Name: projectBy3code
	Depends: mobilesubstrate
	Version: 0.0.1
	Architecture: iphoneos-arm
	Description: An awesome MobileSubstrate tweak!
	Maintainer: gyjrong
	Author: 3code
	Section: Tweaks

其中：

- Package字段用于描述这个deb包的名字，采用的命名方式同bundle identifier类似，均为反向DNS格式，可以按需更改；
- Name字段用于描述这个工程的名字，可以按需更改；
- Depends字段用于描述这个deb包的“依赖”。“依赖”指的是这个程序运行的基本条件，可以填写固件版本或其他程序，如果当前iOS不满足“依赖”中定义的条件，则此tweak无法正常运行。如
- Depends: mobilesubstrate, firmware (>= 8.0)
表示当前iOS版本必须在8.0以上，且必须安装CydiaSubstrate，才能正常运行这个tweak，可以按需更改。
- Version字段用于描述这个deb包的版本号，可以按需更改；
- Architecture字段用于描述deb包安装的目标设备架构，不要更改；
- Description字段是deb包的简单介绍，可以按需更改；
- Maintainter字段用于描述deb包的维护人，例如BigBoss源中所有deb包的维护人均为BigBoss，而非软件作者，可以按需更改；
- Author字段用于描述tweak的作者（注意与Maintainer的区别），可以按需更改；
- Section字段用于描述deb包所属的程序类别，不要更改。

control文件中可以自定义的字段还有很多，但上面这些信息就已经足够了。更全面的说明可以参阅debian的官方网站（http://www.debian.org/doc/debian-policy/ch-controlfields.html）或留意其他deb包里的control文件。值得注意的是，Theos在打包deb时会对control文件作进一步处理，上面的control文件在得到处理后内容变为：

```
原始：
“Package: com.iosre.projectBy3code
Name: projectBy3code
Depends: mobilesubstrate”
“Version: 0.0.1
Architecture: iphoneos-arm
Description: An awesome MobileSubstrate tweak!
Maintainer: snakeninny
Author: snakeninny
Section: Tweaks”

处理后：
Package: com.iosre.projectBy3code
Name: projectBy3code
Depends: mobilesubstrate
Architecture: iphoneos-arm
Description: An awesome MobileSubstrate tweak!
Maintainer: snakeninny
Author: snakeninny
Section: Tweaks
Version: 0.0.1-1
Installed-Size: 104
```

这里Theos更改了Version字段，用以表示Theos的打包次数，方便管理；

增加了一个Installed-Size字段，用以描述deb包安装后的估算大小，可能会与实际大小有偏差，但不要更改。
control文件中的很多信息直接体现在Cydia中，如下所示，大家可以对比看看。


## 5、projectBy3code.plist

这个plist文件的作用和App中的Info.plist类似，它记录了一些配置信息，描述了tweak的作用范围。我们可以用plutil，也可以用Xcode来编辑它。

projectBy3code.plist的最外层是一个dictionary，只有一个名为“Filter”的键,“Filter下是一系列array，可以分为三类。
<!--
![](https://ooo.0o0.ooo/2016/11/01/58182ee3ac4ce.jpeg)-->
![](https://ooo.0o0.ooo/2016/11/01/58182f3cc349a.jpeg)
![](https://ooo.0o0.ooo/2016/11/01/58182f616c699.jpeg)
![](https://ooo.0o0.ooo/2016/11/01/58182f80b74e4.jpeg)

- Bundles，指定若干bundle为tweak的作用对象：

如上配置：tweak的作用对象是三个bundle，即SMSNinja、AddressBook.framework和SpringBoard。

- Classes，指定若干class为tweak的作用对象：

tweak的作用对象是三个class，即NSString、SBAwayController和SBIconModel。

- Executables，指定若干可执行文件为tweak的作用对象：

tweak的作用对象是三个可执行文件，即callservicesd、imagent和mediaserverd。

这三类array可以混合使用：

![](https://ooo.0o0.ooo/2016/11/01/58182f9637bfa.jpeg)

注意，当Filter下有不同类的array时，需要添加一个“Mode：Any”键值对。当Filter下的array只有一类时，不需要添加“Mode：Any”键值对。”







 



