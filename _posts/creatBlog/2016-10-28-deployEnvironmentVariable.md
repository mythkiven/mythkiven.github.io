I---
layout: post
title:  "OS X 部署环境变量"
categories: OS_X
tags:  sublime 环境变量
author: 3code
---

* content
{:toc}


## 1、设置环境变量

##### 1.1 查看本机使用的Shell

```
$ echo $SHELL
输出的是：csh或者是tcsh，就是C Shell。
输出的是：bash，sh，zsh，就是Bourne Shell的一个变种。
```
  如果是Bourne Shell。
那么你可以把你要添加的环境变量添加到你主目录下面的.profile或者.bash_profile，如果存在没有关系添加进去即可，如果没有生成一个。

##### 1.2 Mac可配置环境变量的位置

Mac系统的环境变量，加载顺序为：

- 1、/etc/profile 全局公有配置，系统级别启动加载。【不建议修改】
- 2、/etc/paths 可以全局修改配置，系统级别启动加载。添加path一行一个路径

以上不管是哪个用户，登录时都会读取该文件。下面几个是当前用户级的环境变量，按照从前往后的顺序读取。如果~/.bash_profile文件存在，则后面的几个文件就会被忽略不读了，如果~/.bash_profile文件不存在，才会以此类推读取后面的文件

- 3、~/.bash_profile  一般在这个文件中添加用户级环境变量。每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!
- 4、~/.bash_login
- 5、~/.profile

其他：

- ~/.bashrc没有上述规则，它是bash shell打开的时候载入的。
- /etc/bashrc 一般在这个文件中添加系统级环境变量，全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件。

##### 1.3 设置PATH的语法：

```
export PATH=$PATH:<PATH 1>:<PATH 2>:<PATH 3>:------:<PATH N>
```
 
## 2、实战 配置sublime的环境变量

如果是sublime 2 则应该通过下面命令可以打开subl:

```
$ open /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl
```

然后我们在/usr/local/bin 里面做一个软连接：

```
$ ln -s /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl /usr/local/bin/sublime
```

最后加入环境变量：

```
$ open ~/.bash_profile
添加如下代码：
export PATH=/usr/local/bin:$PATH
```

想立即生效请运行：

```
$ source ~/.bash_profile
```
OK，可以通过终端操控subl了！



