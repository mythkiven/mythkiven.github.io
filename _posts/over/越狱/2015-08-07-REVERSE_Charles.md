---
layout: post
title:  "IOS逆向开发系列_Charles_iPhone网络抓包"
categories:  IOS逆向开发
tags:     Charles
author: 3行代码
---

* content
{:toc}

## 安装Charles

官网下载：[https://www.charlesproxy.com/](https://www.charlesproxy.com/)

从网上找到最新的破解文件：charles.jar，替换系统的即可破解

## 配置



1、配置代理：

    Proxy->Proxy Settings 设置端口8888，勾选Enable transparent HTTP proxying

2、配置手机：
查看电脑IP(Help->Local IP Address)，然后iPhone上也设置相同的IP(在无线局域网->当前连接的wifi->详情->最底部HTTP 代理->手动->填上电脑的IP、端口号8888)

3、拦截HTTPS配置：

    HELP-> SSL Proxying->ICRC 安装证书 

4、手机安装证书：

     Help -> SSL Proxying -> Install Charles Root Certificate on a Mobile Device or Remote Browser

在手机浏览器中访问地址：http://charlesproxy.com/getssl，即可打开证书安装的界面，安装完证书后，就可以截取手机上的Https通讯内容了

5、对于手机上某个请求的网址，右键选择激活SSL proxy





