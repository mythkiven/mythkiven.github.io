---
layout: post
title:  "python3.5 爬虫那些事之phantomjs"
categories:  phantomjs
tags: phantomjs
author: 3行代码
---

* content
{:toc}

### 爬虫简介

python 中主流的爬虫模块简介如下:

- Htmllib（sgmllib）这个模块是非常古老的一个模块,我就不介绍了.
- BeautifulSoup(bs4)这个模块解析html非常专业，具有很好的容错性，可以搜索任意标签，自带编码处理方案。
- Selenium，自动化web测试方案解决者，自带了js解释器，selenium配合浏览器可以用来做动态网页的爬取\分析\挖掘,缺点是耗性能,抓取请求的原理和 fiddler 类似都是基于代理的.
- Scrapy：一个专业的爬虫框架（单机），有相对完整的解决方案。
- API爬虫：这里大概都是需要付费的爬虫API，比如google，twitter的解决方案等

笔者在文章中只会出现前三种方式来做爬虫编写。

### PhantomJS简介

##### PhantomJS 使用场景

**无需浏览器的Web测试：** 无需浏览器的情况下进行快速的Web测试，且支持很多测试框架，如YUI Test、Jasmine、WebDriver、Capybara、QUnit、Mocha等。

**页面自动化操作：**使用标准的DOM API或一些JavaScript框架（如jQuery）访问和操作Web页面。

**屏幕捕获：**以编程方式抓起CSS、SVG和Canvas等页面内容，即可实现网络爬虫应用。构建服务端Web图形应用，如截图服务、矢量光栅图应用。

**网络监控：**自动进行网络性能监控、跟踪页面加载情况以及将相关监控的信息以标准的HAR格式导出。




### 安装

```shell
$ brew update && brew install phantomjs

# 安装 python 模块:selenium
$ pip install selenium

```

### 基本操作

``` shell
$ phantomjs
# 退出phantomjs
phantomjs> phantom.exit() 
```










