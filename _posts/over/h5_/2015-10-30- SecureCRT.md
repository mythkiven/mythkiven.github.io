---
layout: post
title:  "SecureCRT小教程"
categories: OS 
tags:  OS 
author: 3行代码
---

* content
{:toc}

在window上可以通过Xshell调试网络、查看服务器日志，在Mac上没有Xshell的，最好的替换方案就是：SecureCRT。

#### 安装

 google安装

#### 配置

设置UTF8：
Options—>session Options—>Appearance 点选设置UTF8，如下图:

![](https://ooo.0o0.ooo/2016/12/02/584148ea55d87.png)

设置链接：
![](https://ooo.0o0.ooo/2016/12/02/584149554189b.png)
![](https://ooo.0o0.ooo/2016/12/02/5841498e5ab3d.png)

#### 使用

和终端差不多，及时展示当前的请求日志(crt-root.log)使用命令：
 # tail -f crt-root.log

 实际效果如下：可以及时查看服务器日志啦：

![](https://ooo.0o0.ooo/2016/12/02/584149b24a478.png)


#### 后记

一个人开发真的很屌啊，所有的工具都会用。








 



