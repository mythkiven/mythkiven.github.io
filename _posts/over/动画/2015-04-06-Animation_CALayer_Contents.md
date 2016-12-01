---
layout: post
title:  "iOS动画系列-基础部分-CADisplayLink"
categories: iOS动画  
tags: iOS动画 CADisplayLink
author: 3行代码
---

* content
{:toc}

**本系列是个人iOS动画学习笔记，参阅了[iOS Core Animation](https://www.amazon.com/iOS-Core-Animation-Advanced-Techniques-ebook/dp/B00EHJCORC/ref=sr_1_1?ie=UTF8&qid=1423192842&sr=8-1&keywords=Core+Animation+Advanced+Techniques)以及万能的google**



###  1、CADisplayLink

##### 1、是什么

先上文档链接:[https://developer.apple.com/reference/quartzcore/cadisplaylink](https://developer.apple.com/reference/quartzcore/cadisplaylink)

是什么，文档里面已经写的很清楚了:A CADisplayLink object is a timer object that allows your application to synchronize its drawing to the refresh rate of the display.

```
CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self selector:@selector(rotating)];
 [link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
```
##### 2、怎么用

##### 3、是什么