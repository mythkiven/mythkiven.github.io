---
layout: post
title:  "IOS逆向开发系列7_IOS理论基础"
categories: IOS逆向开发  
tags: IOS逆向开发理论基础
author: 3code
---

* content
{:toc}



## 1、简介

Objective-C相关的iOS逆向理论基础

Objective-C语言是一门面向对象的高级语言，想必大家都能较为熟练地掌握它的基本用法，在逆向工程的入门阶段采用Objective-C语言有助于大家更平稳地从App开发进阶到逆向工程。幸运地是，iOS采用的文件格式Mach-O中包含了足够多的原始数据，让我们能够用class-dump等工具还原出二进制文件的头文件；有了这些信息，就可以开始Objective-C级别的逆向工程了，而撰写tweak无疑是这个阶段最受欢迎的项目，下面就从它开始吧！”

##### 1.1 tweak在Objective-C中的工作方式

在第3章中介绍Theos时，已经介绍了tweak的概念。依据维基百科的定义，tweak指的是对电子系统进行轻微调整来增强其功能的工具；在iOS中，tweak特指那些能够增强其他进程功能的dylib，是越狱iOS的最重要组成部分。

正是因为tweak的存在，越狱iOS用户才能依照自己的喜好打造独一无二的个性化系统，iOS开发者才有机会站在优秀软件的肩膀上为它们添砖加瓦，丰富它们的功能，而这些便利都是原版iOS和”
 
 “每次越狱发布后，都会有人把最新的头文件发布出来，Google一下“iOS private headers”即可轻松找到下载链接，省去了自己class-dump的麻烦。
 
 待续...

 