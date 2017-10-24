---
layout: post
title:  "Sublime Text进阶教程"
categories: OS
tags:  OS 
author: 3code
---

* content
{:toc}

![](https://ooo.0o0.ooo/2016/10/29/58147e0b8e24a.jpg)

## 说明

**本人目前sublime2\3都在使用，以下涵盖两个版本的内容，环境是OS X EI。建议大家使用sublime3，下边所述部分插件在2上可能有差异。**
如有问题，请参考[基础教程](http://3code.info/2016/10/29/useSublime/)

## 1、终端模式

Sublime Text提供了终端打开文件的功能：Sublime Text 2的终端命令为sublime,但需要注意的是subl命令默认不在环境变量里，所以我们需要将其添加到环境变量，subl的位置为/Applications/Sublime Text 2.app/Contents/SharedSupport/bin(区分 2 3)),我们需要将这个路径添加到.bash_profile文件里。。

参考文章：[配置环境变量](http://3code.info/2016/10/28/deployEnvironmentVariable/)
完成上述操作后，我们就可以在终端使用Sublime Text 打开文件了：

- sublime fileName      //创建文件
- sublime folderName	 //打开文件夹
- sublime .				 //打开当前目录

## 2、快捷键
如果有冲突或者自定义快捷键，请打开：
preference../Key Bindings 在这里可以修改快捷键设置

##### 2.1选择类

- cmd+D 选中光标所占的文本，继续操作则会选中下一个相同的文本。
- Alt+F3 选中文本按下快捷键，即可一次性选择全部的相同文本进行同时编辑。举个栗子：快速选中并更改所有相同的变量名、函数名等。
- cmd+L 选中整行，继续操作则继续选择下一行，效果和 Shift+↓ 效果一样。
- cmd+Shift+L 先选中多行，再按下快捷键，会在每行行尾插入光标，即可同时编辑这些行。
- cmd+Shift+M 选择括号内的内容（继续选择父括号）。举个栗子：快速选中删除函数中的代码，重写函数体代码或重写括号内里的内容。
- cmd+M 光标移动至括号内结束或开始的位置。
- cmd+Enter 在下一行插入新行。举个栗子：即使光标不在行尾，也能快速向下插入一行。
- cmd+Shift+Enter 在上一行插入新行。举个栗子：即使光标不在行首，也能快速向上插入一行。
- cmd+Shift+[ 选中代码，按下快捷键，折叠代码。
- cmd+Shift+] 选中代码，按下快捷键，展开代码。
- cmd+K+0 展开所有折叠代码。
- cmd+← 向左单位性地移动光标，快速移动光标。
- cmd+→ 向右单位性地移动光标，快速移动光标。
- shift+↑ 向上选中多行。
- shift+↓ 向下选中多行。
- Shift+← 向左选中文本。
- Shift+→ 向右选中文本。
- cmd+Shift+← 向左单位性地选中文本。
- cmd+Shift+→ 向右单位性地选中文本。
- cmd+Shift+↑ 将光标所在行和上一行代码互换（将光标所在行插入到上一行之前）。
- cmd+Shift+↓ 将光标所在行和下一行代码互换（将光标所在行插入到下一行之后）。
- cmd+Alt+↑ 向上添加多行光标，可同时编辑多行。
- cmd+Alt+↓ 向下添加多行光标，可同时编辑多行。

##### 2.2 编辑类

- cmd+J 合并选中的多行代码为一行。举个栗子：将多行格式的CSS属性合并为一行。
- cmd+Shift+D 复制光标所在整行，插入到下一行。
- Tab 向右缩进。只对光标后（或者选中的）的代码有效
- Shift+Tab 向左缩进。
- cmd+[ 向左缩进。对整行有效
- cmd+] 向右缩进。对整行有效
- cmd+K+K 从光标处开始删除代码至行尾。按住cmd，按两次K。
- cmd+Shift+K 删除整行。
- cmd+/ 注释单行。
- cmd+Shift+/ 注释多行。
- cmd+K+U 转换大写。
- cmd+K+L 转换小写。
- cmd+Z 撤销。
- cmd+Y 恢复撤销。
- cmd+U 软撤销，感觉和 Gtrl+Z 一样。
- cmd+F2 设置书签，F2切换书签
- cmd+T 左右字母互换。
F6 单词检测拼写

##### 2.3 搜索类

- cmd+F 打开底部搜索框，查找关键字。
- cmd+shift+F 在文件夹内查找，与普通编辑器不同的地方是sublime允许添加多个文件夹进行查找，略高端，未研究。
- cmd+P 打开搜索框。举个栗子：1、输入当前项目中的文件名，快速搜索文件，2、输入@和关键字，查找文件中函数名，3、输入：和数字，跳转到文件中该行代码，4、输入#和关键字，查找变量名。
- cmd+G 打开搜索框，自动带：，输入数字跳转到该行代码。举个栗子：在页面代码比较长的文件中快速定位。
- cmd+R 打开搜索框，自动带@，输入关键字，查找文件中的函数名。举个栗子：在函数较多的页面快速查找某个函数。
- cmd+： 打开搜索框，自动带#，输入关键字，查找文件中的变量名、属性名等。
- Esc 退出光标多行选择，退出搜索框，命令框等。
- cmd+Shift+P 打开命令框。场景栗子：打开命名框，输入关键字，调用sublime text或插件的功能，例如使用package安装插件。

##### 2.4显示类

- cmd+Tab 按文件浏览过的顺序，切换当前窗口的标签页。
- cmd+PageDown 向左切换当前窗口的标签页。
- cmd+PageUp 向右切换当前窗口的标签页。
- Alt+cmd+1 窗口分屏，恢复默认1屏（非小键盘的数字）
- Alt+ cmd +2 左右分屏-2列
- Alt+ cmd +3 左右分屏-3列
- Alt+ cmd +4 左右分屏-4列
- Alt+ cmd +5 等分4屏
- Alt+ cmd +8 垂直分屏-2屏
- Alt+ cmd +9 垂直分屏-3屏
- cmd+K+B 开启/关闭侧边栏。
- F11 全屏模式
- Shift+F11 免打扰模式

##### 2.5 多重选择（Multi-Selection）

多重选择功能允许在页面中同时存在多个光标，让很多本来需要正则表达式、高级搜索和替换才能完成的任务也变得游刃有余了。
激活多重选择的方法有两几种：

按住 cmd 然后在页面中希望中现光标的位置点击。
选择数行文本，然后按下 Shift + cmd + L。
通过反复按下 cmd + D 即可将全文中与光标当前所在位置的词相同的词逐一加入选择，而直接按下 Alt+F3即可一次性选择所有相同的词。
按下鼠标中键来进行垂直方向的纵列选择，也可以进入多重编辑状态。


## 3、无插件不神器

##### 3.1 AdvancedNewFile:创建文件

可以使用快捷键cmd+alt+n,Sublime Text底部会弹出输入框:
只需在这个输入框里输入需要新建的文件名回车即可（可以带路径）。默认情况下文件会存储在当前目录，如果当前没有目录，会存储在用户的home目录。

##### 3.2 SideBarEnhancements:增强的sidebar

当我们用sublime打开一个文件夹时，会在sublime的左侧出现一个sidebar，以此方便我们可以通过点击的方式快速打开文件。但这个sidebar功能很少。SideBarEnhancements的插件可以增强sidebar的功能。打开命令模式－>进入pci界面->输入SideBarEnhancements回车安装即可。

##### 3.3 Emmet
前身是大名鼎鼎的Zen coding,它使用仿CSS选择器的语法来生成代码，大大提高了HTML/CSS代码编写的速度。

调用方法：cmd+E 或者 !

##### 3.4 Sublime​Code​Intel 
支持所有 Komode Editor 支持的代码语言，如：JavaScript, Mason, XBL, XUL,..

调用方法：跳转到定义位置（Alt+Click 或 Control+Windows+Alt+Up）、返回（Control+Windows+Alt+Left）

##### 3.5 Browser Refresh 
通过一个快捷键可以实现保存文件，切换到浏览器并自动刷新浏览器来查看更改结果。

调用方法：默认为cmd+Shift+R，可自行修改

##### 3.6 ConvertToUTF8
解决编码问题，直接在菜单栏中可以转了，专为中文设计

##### 3.7 Alignment
代码对齐，如写几个变量，选中这几行，cmd+Alt+A，对齐

##### 3.8 Theme – Soda:
完美的编码主题，用过的都说好，Setting user里面添加”theme”: “Soda Dark.sublime-theme”


## 4、Markdown
所需插件：

- MarkDownEditing:在Sublime中编辑MarkDown文件。
- MarkDownPreview:在浏览器中预览MarkDown文件效果。
- Markdown Extended + Extends Monokai：不错的Markdown主题，支持对多种语言的高亮

##### 4.1 设置目录

在文档最前面添加[TOC]，在左侧自动生成目录。

##### 4.2 编辑：

新建一个文档:cmd + N 
语法高亮:cmd + Shift + P:输入ssm 后回车(Set Syntax: Markdown)

#####  4.3 在浏览器预览Markdown文档

Markdown Preview较常用的功能是preview in browser和Export HTML in Sublime Text，前者可以在浏览器看到预览效果，后者可将markdown保存为html文件。
按cmd + Shift + P
输入mp 后回车(Markdown Preview: current file in browser)
此时就可以在浏览器里看到刚才编辑的文档了;

##### 4.4 打印成pdf

将markdown转换为pdf应该有很多种方法的。可直接用谷歌浏览器虚拟打印功能生成。
利用Markdown Preview的Preview in Browser功能可以在浏览器上看到html效果。在页面右键->打印->另存为pdf->调节页边距即可将pdf文件下载下来。


##### 4.5 关于快捷键

- cmd+B 生成本地html文档

st支持自定义快捷键，markdown preview默认没有快捷键，我们可以自己为preview in browser设置快捷键。
方法是在Preferences -> Key Bindings User打开的文件的中括号中添加以下代码(可在Key Bindings Default找到格式)：

```
{ "keys": ["alt+m"], "command": "markdown_preview", "args": { "target": "browser"}
```
"alt+m"可设置为您自己喜欢的按键。


## 5、主题

在设置一个新的主题时，需要设置theme和color_scheme两个方面，前者决定了打开不同类型文件的配色，后者决定了Tab栏，SideBar大小和图标，以及相应字体的大小设置。

软件自带：在Preferences->Color Theme中可以任意选择，选择完即可查看主题风格。并且会在Preferebces.sublime-seetings中自动保存设定。

- ColorSublime：
一个Sublime主题的网站
- Predawn：
一款为Sublime和Atom打造的暗色主题，可以定义Tab的大小，SideBar大小，Find栏大小，为Markdown高亮着色，并提供主题同款的ICON。
- SpaceGray

## 6、Git

- SublimeGit：Git必备
- Gitignore：
一键生成the collection of gitignore boilerplates by Github，多种文件类型任你选，以下用Gitignore新建C语言的忽略文件模板。
- Git Config
设置.gitignore和.gitconfig等文件语法高亮。
- Github Tools / Sublime GitHub：与GitHub网站紧密联系，可以直接在Sublime中打开与GitHub关联的网址
- Git Conflict Resolver：用以解决在Merge过程中产生的冲突用

## 7、Evernote

##### 7.1 相关插件

- Git:方便我们在Sublime中提交博客。
- MarkDownEditing:在Sublime中编辑MarkDown文件。
- MarkDownPreview:在浏览器中预览MarkDown文件效果。
- SublimeTmpl:可以生成MarkDown文件模板。
- Evernote:保存文件到印象笔记.

##### 7.2关联Evernote

在安装好上述插件后，需要将Sublime与我们的印象笔记关联，步骤如下：

1、国内用户打开 [https://app.yinxiang.com/api/DeveloperToken.action](https://app.yinxiang.com/api/DeveloperToken.action) 

国际用户打开 [https://www.evernote.com/api/DeveloperToken.action](https://www.evernote.com/api/DeveloperToken.action) 。

点击Revoke your developer token 授权应用

2、打开 Preferences > Package Settings > Evernote >Settings - User

3、粘贴即可(格式英文)： 

```
{ "noteStoreUrl": "你自己的URL", "token": "你的Token" }
```

##### 7.3测试是否授权成功

接下来我们可以尝试是否授权成功，通过shift+cmd+p打开命令窗口,输入Evernote，就会看见Evernote的许多命令。
可以点击Evernote:list recent notes,如果看到罗列出最新的笔记，则说明授权成功。

##### 7.4 修改模板

SublimeTmpl插件为多种文件格式提供了模板,但很可惜它没有提供md格式的模板，这就需要我们来自定义模板，首先在Users/用户名/Library/Application\ Support/Sublime\ Text\ 3/Packages/SublimeTmpl\templates目录下新建md.tmpl文件，里面填写自己的模板。参考：

```

---
layout: post
keywords: k
description: d
title: t
categories: [笔记]
tags: [笔记]
group: archive
icon: globe
---

```


##### 7.5 储存到印象笔记

打开命令模式，搜索ese,就会找到Evernote send to Evernote as new note命令。

点击后会显示让我们选择存放在哪个笔记本，我们选择后就会自动存放到我们的印象笔记中。

我们再去我们的印象笔记看一下，你会发现存储在印象笔记中的格式已经是排版过的效果。



 



