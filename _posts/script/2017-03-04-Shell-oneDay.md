---
layout: post
title:  "Shell 脚本"
categories:  Shell 
tags: Shell
author: 3行代码
---

* content
{:toc}



>
> 作为一名 iOS 老司机,工作中最常用的就是脚本\命令行了,但是没系统的看过. python 以前用的多,现在要系统的看看 shell ,虽然不一定会学的深入,但是绝对可以进一步提高我的工作效率.
>
> 

说起脚本,就得说脚本的通用性,说起通用性,就得说常见的操作系统: OS X \ Linux \ Windows \ Unix
简单介绍下异同:

- BellLAB 发明了 Unix (还有 C);
- Unix 的 GUI 是 X Window(在内核外跑的应用),包括X Server(与内核通信)与X Client(负责显示)两部分;
- OS X 的 GUI 是自己开发的 Aqua(嵌入内核中) ,而内核则是基于Darwin(Unix + BSD);
- OS X 是全新设计的,和 OS (Mach内核)不同, OS 不是基于 Unix 的;
- Linux 一开始就是免费开源的,基于 Unix.

以上是 Unix 派的,下边的 Windows 则是 DOS + NT 系的:

- Windows 则是 DOS + NT 内核;
- Windows 使用反斜杠: \ 分割路径,原因在于其来自 DOS 最初支持软盘,使用斜杠: /  表示命令参数起始符号
- Unix 使用斜杠:/分割路径

扯的有点远了...

下边回归正文:


### shell 基础 

最初的 Unix shell是在20世纪70年代中期由Stephen R. Bourne写的，现在 shell 分为两种:

- Bourne shell : 默认提示符是$字符。


> Bourne Shell 的各种子类别如下：

>> Bourne Shell （sh）、 Korn shell （ksh）、 Bourne Again（bash）、POSIX shell（sh）

- C shell: 则默认提示符是％字符。

> C shell 子类别如下:

>> C shell（csh）、 TENEX / TOPS C shell（tcsh）


**Bourne shell是在UNIX系统上出现的第一个shell，因此它被称为“shell”。**
Bourne shell通常在大多数UNIX版本上安装为/ bin / sh 




### 热身小例子

``` shell
#!/bin/sh                # 制定脚本解释器,使用 /bin/sh  作为解释器
cd ~                     # 切换到当前用户的 home 目录
mkdir shell_txt          # 创建一个 shell_txt 文件夹
cd shell_txt             # 进入 shell_txt 文件夹

for ((i=0; i<10; i++)); do     # 循环10次
    touch txt_$i.txt           # 创建10个文件:test_1…10.txt文件
done                           # 结束循环

```

- 上边的脚本就创建了 shell_txt 文件夹,并创建10个 txt 文件.
-  cd, mkdir, touch都是系统自带的程序，一般在/bin或者/usr/bin目录下
- for, do, done是sh脚本语言的关键字。

### 1. shell 概念

上边是入门热身,下边解释概念:

> 1. shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务
>
> 2. shell脚本（shell script），是一种为 shell 编写的脚本程序
>
> 3. 业界所说的shell通常都是指shell脚本
> 
> 4. shell编程跟java、php编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。
> 
> 5. 当前主流的操作系统都支持shell编程

- Linux 默认安装就带了 shell 解释器
- Mac OS 不仅带了 sh、bash 这两个最基础的解释器，还内置了 ksh、csh、zsh 等不常用的解释器。
- windows 出厂时没有内置 shell 解释器，需要自行安装

### 2. 脚本解释器

- sh   即Bourne shell，POSIX 标准的shell解释器，它的二进制文件路径通常是/bin/sh，由Bell Labs开发

- bash  是Bourne shell的替代品，属GNU Project，二进制文件路径通常是/bin/bash。

- 业界通常混用bash、sh、和shell，比如你会经常在招聘运维工程师的文案中见到：熟悉Linux Bash编程，精通Shell编程


### 3. 脚本语言

理论上讲，只要一门语言提供了解释器（而不仅是编译器），这门语言就可以胜任脚本编程，常见的解释型语言都是可以用作脚本编程的，如：Perl、Tcl、Python、PHP、Ruby。Perl是最老牌的脚本编程语言了，Python这些年也成了一些linux发行版的预置解释器。

编译型语言，只要有解释器，也可以用作脚本编程，如C shell是内置的（/bin/csh），Java有第三方解释器Jshell，Ada有收费的解释器AdaScript。

shell 只定义了一个非常简单的编程语言，如果你的脚本程序复杂度较高，或者要操作的数据结构比较复杂，那么还是应该使用Python、Perl这样的脚本语言，或者是你本来就已经很擅长的高级语言。因为sh和bash在这方面很弱，比如说：

如果你的脚本是提供给别的用户使用，使用sh或者bash，你的脚本将具有最好的环境兼容性，perl很早就是linux标配了，python这些年也成了一些linux发行版的标配，至于mac os，它默认安装了perl、python、ruby、php、java等主流编程语言。



### 4. 编写脚本


打开文本编辑器，新建一个文件，扩展名为sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用php写shell 脚本，扩展名就用php好了。

输入一些代码，第一行一般是这样：

``` shell 
#!/bin/bash
#!/usr/bin/php
```

“#!”是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行。

### 5. 运行脚本

运行Shell脚本的方法：

- 作为解释器参数

在终端,直接运行解释器 :

``` shell 
$/bin/sh test.sh
$/bin/php test.php
```

这种方式运行的脚本，不需要在第一行指定解释器信息.

- 或者

``` shell 
$sh test.sh
```


**说明,以下仅挑选部分语法知识,更多的请参见文末的链接,里面内容很全的.**



### 6. 变量

一个变量只不过是一个指向实际数据的指针

##### 定义变量

定义变量时,变量名不加美元符号（$），如：

```
# 标量变量:
your_name="qinjx"
# 只读变量
NAME="Zara Ali"
readonly NAME
```

##### 变量类型

- 局部变量  一个局部变量是一个变量，只是shell的当前实例中存在。

- 环境变量 环境变量可用于shell的任何子进程。一些程序需要环境变量才能正常工作。

- Shell变量 - shell变量是一个特殊的变量，由shell设置，是shell所需要的，以便正确地运行。

注意: 变量名和等号之间不能有空格


##### 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号即可，如：

``` shell 
your_name="qinjx"
echo $your_name
echo ${your_name}
```
变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界,比如下边的例子
 
``` shell
for skill in Ada Coffe Action Java; do
    echo "I am good at ${skill}Script"
done 
```

如果不给skill变量加花括号，写成echo "I am good at $skillScript"，解释器就会把$skillScript当成一个变量（其值为空），代码执行结果就不是我们期望的样子了

##### 重定义变量

已定义的变量，可以被重新定义，如：

``` shell 
your_name="qinjx"
echo $your_name

your_name="alibaba"
echo $your_name
```

这样写是合法的，但注意，第二次赋值的时候不能写$your_name="alibaba"，使用变量的时候才加美元符。

##### 注释

以“#”开头的行就是注释，会被解释器忽略。

多行注释

sh里没有多行注释，只能每一行加一个#号


如果在开发过程中，遇到大段的代码需要临时注释起来，过一会儿又取消注释，怎么办呢？每一行加个#符号太费力了，可以把这一段要注释的代码用一对花括号括起来，定义成一个函数，没有地方调用这个函数，这块代码就不会执行，达到了和注释一样的效果。



### 7. 字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了，哈哈），字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。

##### 单引号

	str='this is a string'

单引号字符串的限制：

单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
单引号字串中不能出现单引号（对单引号使用转义符后也不行）

##### 双引号

``` shell 
your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"
```

双引号里可以有变量
双引号里可以出现转义字符


### 8. 字符串操作

##### 拼接字符串

``` shell 
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

echo $greeting $greeting_1
```

##### 获取字符串长度：

```
string="abcd"
echo ${#string} 
#输出：4
```

##### 提取子字符串

```
string="alibaba is a great company"
echo ${string:1:4} 
#输出：liba
```
##### 查找子字符串

```
string="alibaba is a great company"
echo `expr index "$string" is`
#输出：8，这个语句的意思是：找出单词is在这名话中的位置
```
 
### 9. 流程控制

##### if 

```
if condition
then
    command1 
    command2
    ...
    commandN 
fi 
```

写成一行 

` if `ps -ef | grep ssh`;  then echo hello; fi `

末尾的fi就是if倒过来拼写，后面还会遇到类似的


##### if else

```
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi 

```
 

##### for while

```
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done 
```

写成一行：

`for var in item1 item2 ... itemN; do command1; command2… done;`

 
### 10. 函数 

##### 文件包含

可以使用source和.关键字，如：

```
# 写法1
source ./function.sh
# 写法2
. ./function.sh 
```

在bash里，source和.是等效的，他们都是读入function.sh的内容并执行其内容（类似PHP里的include），为了更好的可移植性，推荐使用第二种写法。

包含一个文件和执行一个文件一样，也要写这个文件的路径，不能光写文件名，比如上述例子中:

```
. ./function.sh
# 不可以写作：

. function.sh 
```

如果function.sh是用户传入的参数，如何获得它的绝对路径呢？方法是：
```
#$1是用户输入的参数，如function.sh 
real_path=`readlink -f $1` 
. $real_path 

```




### 11. 终端\shell 常用命令

- `echo` 显示在屏幕上

$  echo "wwwww"
终端就会打印: wwwww

- 获取文件的绝对路径 

```
# function.sh 的绝对路径: real_path
real_path=`readlink -f $1` 
```

- ps 查看进程表 

```
$ ps 
  PID TTY           TIME CMD
75831 ttys000    0:00.13 -bash
75832 ttys001    0:00.13 -bash
78258 ttys001    2:07.88 ruby /Users/zzz/.rvm/gems/ruby-2.3.0/bin/jeky
80088 ttys001    0:00.04 /Users/zzz/.rvm/gems/ruby-2.3.0/gems/rb-fseve
42208 ttys002    0:00.29 -bash
```
- date 打印当前日期

```
$ date
2017年 3月 4日 星期六 21时03分15秒 CST
```

- pwd 打印当前路径

- ls 列出当前路径的内容 


### 12. OS X 常用终端命令

### 13.更多参考链接 

- [OS X Support Essentials 10.11](http://training.apple.com/pdf/support-10.11-exam-prep.pdf)
- [高级Bash脚本指南](http://tldp.org/LDP/abs/html/opprecedence.html)
- [高级Bash脚本指南](http://tldp.org/LDP/abs/html/)
- [Unix   Shell教程 英语](http://www.tutorialspoint.com/unix/unix-shell.htm)
- [Linux  Shell教程 英语](https://bash.cyberciti.biz/guide/Main_Page)
- [Unix 入门教程](https://www.tutorialspoint.com/unix/unix-file-permission.htm)
