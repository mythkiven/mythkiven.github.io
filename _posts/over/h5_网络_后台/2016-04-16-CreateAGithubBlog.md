---
layout: post
title:  "静态页面博客:jekyll+GitHubPages"
categories: 博客 jekyll 
tags:  博客 jekyll 
author: 3code
---

* content
{:toc}

前段时间一朋友想开个独立博客，问我有没有好的教程参考，以前弄博客时写过流程，这又重新整理了下发上来。Mac上想建博客的可以参考下。

![](https://ooo.0o0.ooo/2016/10/21/5809801186a24.jpg)


## 说明
1、以下系统基于OS X EI;

2、免费主机：github + 静态制作：jekyll

## 1、git\github

#### 1.1 github知识普及：

githubPages 是基于github开源库的，所有博客的内容源码都是可以被其他人看到的，敏感数据不能放在pages上。在提交github时需要注意。

个人Pages页面 实际上是存在GitHubPages的开源库中，需要使用用户名来命名这个库，比如 mythkiven.github.io。可以在master分支上构建和发布你的 GitHub Pages 网页。通过Automatic Page Generator 可以自动的构建一个页面。当用户 Pages 构建完之后，打开http(s)://mythkiven.github.io就可以正常使用了。

#### 1.2  注册下载Git客户端

注册Github账号；由于需要一个本地端来进行git操作，最好使用命令行，客户端的话推荐：SourceTree。或者用github自带的客户端。

#### 1.3 在github上创建Pages：

[参考官方页面提示](https://pages.github.com/)



## 2、Jekyll

本博客使用Jekyll搭建，好处：

1. 不需要使用额外的数据库；
2. 支持markdown，liquid，以及原始的html、css；
3. 可以定义模板，并在模板上进行代码复用；
4. github对其支持，可以直接在github上搭建。

当然缺点也有:

1. 生成的是静态网页，无法动态加载，若需要外部服务如评论，只能使用类似于disquz，多说这样的外部插件了
2. 仅仅适合个人博客或小型网站，不适合大中型网站
3. 没有数据库及服务端的逻辑

## 3、本地环境搭建


#### 环境搭建

使用Jekyll，需要以下环境: 

1. Ruby；
2. Bundler。

#### 3.1 安装Ruby：

详细的安装文档，可以查看 Ruby 官方的[安装](http://www.ruby-lang.org/en/downloads/)介绍。Mac 下使用 [Homebrew](http://brew.sh/) 来安装，挺方便的。

* 
```
$ brew install ruby
```

* 安装完成之后运行查看版本号，过低需要更新。

* 
```
$ ruby -v  
```

#### 3.2 安装RubyGems、Bundler： 

* <p>下载 RubyGems 安装包，2.0.6 版本：<a href="http://production.cf.rubygems.org/rubygems/rubygems-2.0.6.tgz" target="_blank">tgz</a> - <a href="http://production.cf.rubygems.org/rubygems/rubygems-2.0.6.zip" target="_blank">zip</a> - <a href="http://production.cf.rubygems.org/rubygems/rubygems-update-2.0.6.gem" target="_blank">gem</a> - <a href="http://github.com/rubygems/rubygems" target="_blank">git</a>。安装到本地之后，在终端检查更新：</p>

* 
```
# 可能需要根权限
$ gem update --system
# 检查当前版本
$ gem -v
```

* 可以参考<a href="https://rubygems.org/pages/download" target="_blank">官网安装教程</a>
* gem update --system。这一步需要翻墙，否则会出现404错误。<a href="https://ruby.taobao.org/" target="_blank">解决办法参考</a>

*  安装Bundler
```
    $ gem install bundler
```

#### 3.3 安装 Jekyll


```
# 可能需要根权限
$ gem install jekyll
# 安装完了之后，查看版本号，我的是 jekyll 3.3.0
$ jekyll -v
```
至此，你已经成功在本地电脑上安装好了 Jekyll。

安装过程中可能会遇到的问题：

	# 问题1：cannot load such file -- bundler (LoadError)::::
	# 解决方案：
	$ gem install bundler
	$ bundle install
	$ bundle exec jekyll serve
	# 问题2：服务无法开启：
	$ gem install json 
	# 或者：
	$ jekyll serve --config _config.yml,_config-dev.yml 

#### 3.4 Jekyll的使用

```
# 创建你的博客
$ jekyll new blog  
# 进入blog目录,记得一定要进入创建的目录，否则服务无法开启
$ cd blog    
# 启动本地预览	 
$ jekyll serve 	 

# 如果是其他目录:
$ cd 目录
$ bundle  
$ bundle exec jekyll serve
```

注意，自己创建或者下载的博客都要先看是否有Gemfile文件，没有需要创建的。

创建一个名为Gemfile的文件，并添加如下内容,保存在博客的根目录。

    source 'https://rubygems.org'
    gem 'github-pages', group: :jekyll_plugins

然后运行:

    $ bundle install

具体见下文。

#### 3.5 博客主题

下载地址参考如下：
[jekllthemes](http://jekyllthemes.org/)

这个网站中主题挺多的，可以下载直接使用。如果想改造成自己独特的风格又不懂html，最好还是去看看CSS相关的内容[http://www.w3school.com.cn/](http://www.w3school.com.cn/)，w3school里面都是比较基础而且容易入手的，不懂Html的稍微学点CSS相关知识，就可修改页面风格。
 
下载主题之后:

    $ cd 目录
    $ jekyll build 
    $ jekyll serve
    # 在浏览器中输入 http://localhost:4000 即可预览刚才下载的主题

#### 3.6 博客编辑工具

以上操作之后，就可以在本地预览博客，下边说说编辑博客内容:

免费的编辑工具：

* [mou](http://25.io/mou/)
* [macdown](http://macdown.uranusjr.com/)

###### Mou

集成 Tumblr 和 Scriptogr.am 发布博文；内置 CJK 字符支持。

###### MacDown 特色

代码高亮；MacDown 支持Task list


## 4 jekyll的注意事项

####  4.1 更新

Jekyll 是一个动态开源项目，它会频繁地更新。当服务器更新后，本地就会过时，可能导致你的网站出现本地和发布在 GitHub 的样子不一致。

```
$ bundle 
```

或者 

```
$ gem update github-pages #没安装bundler
```

#### 4.2 Gemfile文件

Gemfile是一个用于描述gem之间依赖的文件。gem是一堆Ruby代码的集合，它能够为我们提供调用。
Gemfile是可通过Bundler创建：

```
$ gem install bundler
$ bundle init
$ bundle install
```
Gemfile文件中设置的内容如下：

```
# 以下必须：
source "https://rubygems.org"
gem "jekyll-paginate"
gem "kramdown"
group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.6"
   
# 以下按需：
gem "jekyll-watch"
gem "wdm", "~> 0.1.0" if Gem.win_platform?
# 以下按需：可以配合 Live Reload Extension 可以自动刷新
gem 'guard'
gem 'guard-jekyll-plus'
gem 'guard-livereload'
```

设置好Gemfile之后，执行 $ bundle install 如果开启了本地页面更新，那么会很耗时间。

B、创建guard配置文件

执行指令，将会生成一个Guardfile文件。

```
$ guard init
```
生成的Guardfile文件内有一些代码，在代码的最后添加如下代码：


```
guard 'jekyll-plus', :serve => true do
  watch /.*/
  ignore /^_site/
end

guard 'livereload' do
  watch /.*/
end
```

C、添加livereload插件

使用chrome安装Live Reload Extension。

D、执行运行指令：

```
$ bundle exec guard start
```
这里注意一下，livereload要先关闭。

运行上面指令，当出现“Guard is now...”，再运行livereload。

然后会出现“connected”连接了，接下来修改内容就会自动刷新页面了。


#### 4.3 配置 Jekyll


可以通过创建一个 _config.yml 来配置 Jekyll 大部分属性。


## 番外：jekyll的使用

#### 1、RubyGems 镜像


<p>RubyGems的淘宝镜像：<a href="https://ruby.taobao.org/">https://ruby.taobao.org/</a>。
<br/>
 Ruby China镜像：<a href="https://gems.ruby-china.org/">RubyGems 镜像- Ruby China</a>。</p>

#### 2、jekyll-paginate 依赖缺失

<p>因为 jekyll 3 中默认安装已经没有这个分页组件了，官方把这个分页组件插件化了，因此要独立安装。详情见 <a href="https://jekyllrb.com/docs/pagination/">https://jekyllrb.com/docs/pagination/</a>。</p>

#### 3、其他可能需要安装的：

```
$ gem install jekyll-paginate
```

#### 4、更换端口：


```
# 查看端口被占用情况
$ netstat -ano  
# 在启动jekyll服务的时候指定端口号:  http://localhost:3000/
$ jekyll serve --port 3000
# 还可以在配置文件_config.yml中添加端口号
```

#### 5、jekyll库中所包含的主要文件及文件夹有：

<ol>
<li>文件夹_layouts：用于存放模板的文件夹，</li>
<li>文件夹_posts：用于存放博客文章的文件夹</li>
<li>文件夹css：存放博客所用css的文件夹,比如主题文件以及高亮文件都是放在此处的</li>
<li>.gitignore：可以删掉，后面会将项目添加到git项目，所以这个不需要了</li>
<li>_coinfig.yml：jekyll的配置文件，里面可以定义相当多的配置参数，具体配置参数可以参照其官网</li>
<li>index.html：项目的首页</li>
<li>_includes:用于存放一些固定的HTML代码段，文件为.html格式，可以在模板中通过liquid标签引入，常用来在各个模板中复用如 导航条、标签栏、侧边栏之类的在每个页面上都一样不变的内容，需要注意的是，这个代码段也可以是未被编译的，也就是说也可以使用liquid标签放在这些代码段中</li>
</ol>
<p>通过修改配置文件_coinfig.yml以及post和layouts就可以实现主要blog的建立和修改。</p>

#### 6、运行jekyll server，提示 Jekyll::Paginate


```
 # 在自己的github仓库里面，运行 jekyll build + jekyll server 报错：Jekyll::Paginate
 # 方案如下：
 # 在Gemfile文中添加：gem 'jekyll-paginate'
 $ bundle
 $ gem install jekyll-paginate
 # OK！

```
 


## 参考

相关内容可参考:

------------ 

 |- [jekyll](http://jekyll.com.cn/)  | - [github](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/) | - [ytysj](http://ytysj.github.io/blog/myblog3)
------------ 

 |- [jianshu](http://www.jianshu.com/p/6c157af09e84 ) |- [ezlippi](http://www.ezlippi.com//blog/2015/03/github-pages-blog.html) | - [waylau](http://waylau.com/jekyll-static-bog/?utm_source=tuicool&utm_medium=referral)| - [wowubuntu](http://wowubuntu.com/markdown/) | - [cnblogs](http://www.cnblogs.com/strick/p/5448570.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
------------ 
