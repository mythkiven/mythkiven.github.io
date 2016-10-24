---
layout: page
title: Collections
permalink: /collection/
icon: bookmark
---

* content
{:toc}
## IOS APP 如何更加安全

### 1、hack必备：只有懂攻击，才能更给力的防御
##### 1.1常用的命令和工具:终端</br>
[原文](http://blog.csdn.net/jiang314/article/details/52883433)<br/>

##### 1.2 使用终端手动编译Hello world
[参考]( http://www.cocoachina.com/industry/20140211/7800.html)

##### 1.3、GDB
--命令行调试器。虽说xcode比较强大，但是gdb可以让你更精确的调试。而且这在很多操作系统里都是可以通用的命令。


### 2、IOS APP 安全系列文章《：：》分析+动态修改APP
##### 2.1、IOS 安全系列 1 介绍如何在越狱的设备上搭建用来测试iOS安全的环境
 [P1](http://resources.infosecinstitute.com/ios-application-security-part-1-setting-up-a-mobile-pentesting-platform/)

##### 2.2、IOS 安全系列2介绍如何利用class-dump-z 和 Clutch 来dump类信息，利用这些信息，可以理解代码的设计和代码内部是如何工作的。
[P2](http://resources.infosecinstitute.com/ios-application-security-part-2-getting-class-information-of-ios-apps/ )

例如有一个方法 -(BOOL)isFacebookSessionValid ，在某种情况下返回 NO，可以在运行时修改 instance variable变量的值，操纵它让它返回YES<br/>

##### 2.3、Understanding the Objective-C Runtime 

IOS 安全系列3 提到Method Swizzling
[P3](http://resources.infosecinstitute.com/ios-application-security-part-3-understanding-the-objective-c-runtime/)
##### 2.4、Runtime Analysis Using Cycript (Yahoo Weather App)
Part4介绍了用Cycript动态分析和修改app的方法。文章拿Yahoo Weather app做的例子。其中一处改动是给加上了badge number。
[P4](http://resources.infosecinstitute.com/ios-application-security-part-4-runtime-analysis-using-cycript-yahoo-weather-app/)
![](http://farm3.staticflickr.com/2823/9241398663_b80335cd23.jpg)

##### 2.5、Part5 分享了一些高级分析技术。分析了如何获得特定类的信息（方法名，实例变量名称），并且如何在运行时修改。
[P5](http://resources.infosecinstitute.com/ios-application-security-part-5-advanced-runtime-analysis-and-manipulation-using-cycript-yahoo-weather-app/)
##### 2.6、New Security Features in IOS 7
[P6](http://resources.infosecinstitute.com/ios-application-security-part-6-new-security-features-in-ios-7/)
##### 2.7、 Installing and Running Custom Applications on Device without a registered developer account 
[P7](http://resources.infosecinstitute.com/ios-application-security-part-7-installing-and-running-custom-applications-on-device-without-a-registered-developer-account/)







[IOS开发之GDB命令行调试](http://blog.sina.com.cn/s/blog_71715bf801016d49.html)


* [ ]( )
* [把第三方 iOS 应用转成动态库 ](http://blog.imjun.net/2016/10/08/%E9%BB%91%E7%A7%91%E6%8A%80%EF%BC%9A%E6%8A%8A%E7%AC%AC%E4%B8%89%E6%96%B9-iOS-%E5%BA%94%E7%94%A8%E8%BD%AC%E6%88%90%E5%8A%A8%E6%80%81%E5%BA%93/)
* [iOS符号表恢复&逆向支付宝 ]( http://blog.imjun.net/2016/08/25/iOS%E7%AC%A6%E5%8F%B7%E8%A1%A8%E6%81%A2%E5%A4%8D-%E9%80%86%E5%90%91%E6%94%AF%E4%BB%98%E5%AE%9D/)
* [ ]( )
* [ ]( )
* [ ]( )
* [IOS 安全博客 ](http://blog.csdn.net/yiyaaixuexi/article/list/1 )


		你的应用正在被其他对手反向工程、跟踪和操作！你的应用是否依旧裸奔豪不防御？
 		郑重声明一下，懂得如何攻击才会懂得如何防御，一切都是为了之后的防御作准备。废话少说，进入正题。	今天总结一下为hack而做的准备工作。


* [IOS层次 based on MVCS and KVO.](https://github.com/southpeak/Minya)

* [【应用号】IDE + 破解 + Demo](https://github.com/gavinkwoe/weapp-ide-crack)
* [ ]( )
* [ ]( )
* [ ]( )
* 
### 代码
 * [高仿Bilibili客户端](https://github.com/MichaelHuyp/Bilibili_Wuxianda)



杨君，中山大学计算机系研究生，iOS 开发者，擅长领域 iOS 安全和逆向工程，个人博客：http://blog.imjun.net 。




## 前端工具

* [box-shadow生成工具](http://www.cssmatic.com/box-shadow)
* [渐变生成器](http://www.cssmatic.com/gradient-generator)
* [CSS3 生成器](http://www.cssreflex.com/css-generators/)
* [在线压缩图片](https://tinypng.com/)
* [图床：图片存到网络](https://sm.ms/)
* [时间戳在线](http://tool.chinaz.com/Tools/unixtime.aspx)

## 前端语言
### JS
* [JavaScript 标准参考教程（alpha） -阮一峰](http://javascript.ruanyifeng.com/)
* [JavaScript Promise迷你书 -azu](http://liubin.org/promises-book/)
* [You Don't Know JS (book series)](https://github.com/getify/You-Dont-Know-JS)
* [You Don't Need jQuery](https://github.com/oneuijs/You-Dont-Need-jQuery/blob/master/README.zh-CN.md)
* [JavaScript 秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/)
* [JavaScript 设计模式 系列 AlloyTeam](http://www.alloyteam.com/2012/10/common-javascript-design-patterns/)

### ES2015
* [http://www.ecma-international.org/ecma-262/6.0/](http://www.ecma-international.org/ecma-262/6.0/)
* [30分钟掌握ES6/ES2015核心内容（上）](http://segmentfault.com/a/1190000004365693)
* [30分钟掌握ES6/ES2015核心内容（下）](http://segmentfault.com/a/1190000004368132)
* [《ECMAScript 6入门》 -阮一峰](https://github.com/ruanyf/es6tutorial)
* [EcmaScript6 全规范（含node） -ouvens](https://github.com/ouvens/es6-code-style-guide)

### NodeJS

* [七天学会NodeJS -Nanqiao Deng](https://nqdeng.github.io/7-days-nodejs)

## 框架&脚手架

### webpack

* [Webpack 中文指南 -赵达](https://www.gitbook.com/book/zhaoda/webpack/details)
* [Webpack傻瓜式指南（一） -前端外刊评论 知乎专栏](http://zhuanlan.zhihu.com/FrontendMagazine/20367175)
* [Webpack傻瓜指南（二）开发和部署技巧 -前端外刊评论 知乎专栏](http://zhuanlan.zhihu.com/FrontendMagazine/20397902)
* [Webpack傻瓜指南（三）和React配合开发 -前端外刊评论 知乎专栏](http://zhuanlan.zhihu.com/FrontendMagazine/20522487)
* [Webpack，101入门体验 -Yika](http://www.html-js.com/article/3009)
* [Webpack 入门指迷 -题叶](https://segmentfault.com/a/1190000002551952)
* [https://webpack.github.io/ Webpack 官网](https://webpack.github.io/)


### Vue

* [awesome-vue](https://github.com/vuejs/awesome-vue)
* [Vue.js 和 Webpack（一） -Randy Lu](http://djyde.github.io/2015/08/29/vuejs-and-webpack-1/)
* [Vue.js 和 Webpack（二） -Randy Lu](http://djyde.github.io/2015/08/30/vuejs-and-webpack-2/)
* [Vue.js 和 Webpack（三） -Randy Lu](http://djyde.github.io/2015/08/31/vuejs-and-webpack-3/)
* [Vuejs 1.0 中文系列视频教程 -Laravist](https://laravist.com/series/vue-js-1-0-in-action-series)
* [Vuejs-QQ群 相关资料](https://github.com/jsfront/src/blob/master/vuejs.md) 来自豪情


### React

* [深入理解 React -Thinking in React 中文版](http://reactjs.cn/react/docs/thinking-in-react.html)
* [Thinking in React](http://facebook.github.io/react/docs/thinking-in-react.html)

### AngularJS

* [学习AngularJS 1.x -Harry<harry@andtoo.net>](https://hairui219.gitbooks.io/learning_angular/content/zh/index.html)
* [AngularJS api 官网](https://docs.angularjs.org/api)
* [AngularJS入门教程——AngularJS中文社区提供](https://github.com/zensh/AngularjsTutorial_cn)
* [AngularJS 教程 \| 菜鸟教程](http://www.runoob.com/angularjs/angularjs-tutorial.html)

### 测试

* [测试框架 Mocha 实例教程 阮一峰](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)

## 类库与插件

* [Masonry](http://masonry.desandro.com/)
* [jssor](http://www.jssor.com/)
* [cssslider](http://cssslider.com/)

## 模块化

* [后端程序员的 JavaScript 之旅 - 模块化（一）](http://lishaopeng.com/2016/02/05/js-module/)
* [后端程序员的 JavaScript 之旅 - 模块化（二）](http://lishaopeng.com/2016/02/11/js-module2/)
* [后端程序员的 JavaScript 之旅 - 模块化（三）](http://lishaopeng.com/2016/02/19/js-module3/)

* [CommonJS 规范 -来自 阮一峰 JavaScript 标准参考教程(alpha)](http://javascript.ruanyifeng.com/nodejs/module.html)

## 技巧

* [将footer固定在页面底部的实现方法](https://segmentfault.com/a/1190000004453249)


## GitBook 及其插件

* [Gitbook 的使用和常用插件 -赵达](http://zhaoda.net/2015/11/09/gitbook-plugins/)
* [gitbook-plugin-expandable-chapters](https://plugins.gitbook.com/plugin/expandable-chapters)

## Chrome 插件

* [Chrome扩展及应用开发 -图灵电子书](http://www.ituring.com.cn/minibook/950)

* [有哪些鲜为人知却非常有意思、好用的 Chrome 扩展？ -知乎](https://www.zhihu.com/question/23228162#answer-28057391)
* [Dribbble New Tab](https://chrome.google.com/webstore/detail/dribbble-new-tab/hmhjbefkpednjogghoibpejdmemkinbn)

    新建 tab 时，显示 dribbble 上的精选作品。

## Comments

{% include comments.html %}
