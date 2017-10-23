---
layout: post
title:  " 持续集成系列 - CI + Jenkins的魔法世界"
categories: CI
tags: fastlane CI
author: 3行代码
---

* content
{:toc}


>
>最近一直在关注移动端的持续集成部署相关的内容,在网搜 Jenkins 资料是很多,但是基本都不是移动端的集成部署.下边将个人的使用体会分享出来,希望能帮助到需要的伙伴们.
>

**在进行 CI 之前,你需要一些基本的知识**

### CI 基础知识

在互联网的产品开发时代,产品迭代越来越频繁,“从功能开发完成直到成功部署”这一阶段被称为软件开发“最后一公里”.很多开发团队也越来越认识到,自动化测试和持续部署可帮助开发团队提高迭代效率和质量.
那么,如何更好地解决“最后一公里”这一问题呢?

可以分为两部分:持续集成和持续部署
- 持续集成:是指整个开发过程,每个人开发的模块向项目主体交付时,频繁进行集成以便更快地发现其中的错误
- 持续部署:是指开发过程中连续的阶段性版本的上线, V1.1.0上线部署,V1.1.1上线部署...
- 前者和后者时间尺度是一样的,不过前者是涵盖每一次 push ,后者是指一个稳定的版本,可以内测和发布用的
- 实际使用中,两者是结合在一起的,持续集成的平台上,可以进一步配置持续部署,这样就可以实现:每次 push 自动构建测试,每次稳定的版本,可以顺利的构建分发用于内测.


##### 持续集成(Continuous Integration)

持续集成:是指整个开发过程,个人开发的模块,向项目主体交付,频繁进行集成以便更快地发现其中的错误.涉及到的内容为:

- 自动化测试:单元测试+ UI 测试
- 持续集成平台:Jenkins (自动化的构建) \ Travis CI (基于 github 平台的CI持续集成)等
- 版本控制:如 Git\SVN\CVS 等
- 部署发布工具:fastlane 等
- 部署发布平台:蒲公英\Fir.im (目前支持 Android 和 iOS 应用的分发,提供的工具有 flow.ci)
- 一般流程(Jenkins 为例):Jenkins 服务器轮训 Git ,发现新的版本,就会 clone 到本地,并按照预先设定的脚本进行自动编译,并执行测试.如果有问题,就会发出邮件通知.

##### 持续部署(Continuous Deployment)

持续部署:是指开发过程中的连续的阶段性版本, V1.1.0上线部署,V1.1.1上线部署...
在移动端主要是使用 fastlane 实现持续部署:

- 开发项目,单元测试
- 模块开发,集成\UI 测试
- Archive项目
- 上传包到iTunes
- 在iTunes后台提交版本截图\描述等信息(也可以在测试完成后配置)
- 配置TestFlight,分发测试Beta版
- 测试,改 bug
- 测试完成,提交App Store审核




##### 1.  Xcode 命令行工具 : xcodebuild

Xcode 不仅可以通过 GUI 编译运行,还可以通过命令行实现编译运行,这也是 CI 的基础条件之一.

官方的使用方法:

``` shell 
xcodebuild [-project <projectname>] [[-target <targetname>]...|-alltargets] [-configuration <configurationname>] [-arch <architecture>]... [-sdk [<sdkname>|<sdkpath>]] [-showBuildSettings] [<buildsetting>=<value>]... [<buildaction>]...

...
```

关键字:
``` 
Options:
    -usage                              print brief usage
    -help                               print complete usage
    -verbose                            provide additional status output
    -license                            show the Xcode and SDK license agreements
    ....
```

**实战:**
```
$ cd  /Users/guoyinjinrong1/Git_Projecrt/XCTest/UITEST 
$ xcodebuild build -workspace UITEST.xcworkspace -scheme UITEST -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```

##### 2. project 项目的命令行编译:

``` shell
# 列出 xcodebuild 所有可用的参数:
$ xcodebuild --help
# project 项目的命令行编译:
$ xcodebuild -project {project}.xcodeproj -target {xxtarget} -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
#
```

使用 iphonesimulator SDK 是为了避免签名错误.直到我们稍后引入证书之前,这一步是必须的.通过设置 ONLY_ACTIVE_ARCH=NO 我们可以确保利用模拟器架构编译工程.你也可以设置额外的属性,例如 configuration,输入 man xcodebuild 查看相关文档.

##### 3.CocoaPods 项目

``` shell 
# 需要用下面的命令来指定 workspace 和 scheme：
$ xcodebuild -workspace {workspace}.xcworkspace -scheme {scheme} -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```

schemes 是由 Xcode 自动生成的,但这在服务器上不会发生.确保所有的 scheme 都被设为 shared 并加入到仓库中.否则它只会在本地工作而不会被  CI 系统识别

##### 4.Xctool 

Xctool 是来自 Facebook 的命令行工具,它可以简化程序的编译和测试.它的彩色输出信息比 xcodebuild 更加简洁直观.同时还添加了对逻辑测试,应用测试的支持.(Travis 中已经预装了 xctool.要在本地测试的话,需要用 Homebrew 安装 xctool)

xctool 用法非常简单,它使用的参数跟 xcodebuild 相同:
``` shell
xctool test -workspace TravisExample.xcworkspace -scheme TravisExampleTests -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```



### Jenkins 简介

1.简介:

- Jenkins 由以前的 hudson 更名而来
- Jenkins 是一个跨平台的开源软件项目,旨在实现持续集成和持续交付以提高工作效率
- Jenkins 的主要功能是监视重复工作的执行,例如项目的构建等
- 最终都是为了提高软件开发的效率

2.特点:

- 易于安装:一个命令就可启动,也方便部署到各种Web容器中（如tomcat)
- 易于配置:所有的配置都在Web界面实现,权限控制得也不错
- 插件支持:基本上所有的扩展都是有插件完成的,开发插件也很方便,由此产生了庞大的社区
- 支持分布式构建:Jenkins能够让通过主从模式（master/slave）多台机器一起构建
- 采用 shell 自定义脚本,控制集成部署环境更加方便灵活
- 构建失败发邮件通知相关人员解决
- 自动按天备份 war 包,Jenkins 配置备份以及版本控制化

3.公司的配置流程:

1\Jenkins服务器一般先架设在一台服务器上,有配置管理人员管理
2\产品有构建需求后,配置管理人员就新建一个任务,配好源码仓库,设置构建时间,指定运行脚本来编译测试产品,并且设置报告输出

4.弊端:

- 多平台或依赖的包:
如果你的软件想在不同的操作系统软件构建并验​​证,这将是相当有难度和技巧性,一般是准备不同的操作系统的机器,然后使用主/从模式进行分布式构建.在C++中你可以使用有关的交叉编译器;当然你可有使用虚拟机的技术[vagrant/virtualbox](http://larrycai.github.com/2011/10/25/vagrant-jenkins-ci.html)
- 配置管理人员的工作:
一般来说Jenkins的构建任务,它是集中控制的.大多来说都是有专职的配置管理人员来管理,否者权限会混乱.
当有需求时,他们负责在CI服务器创建任务（这些配置文件不是有版本控制的）,这个就涉及到了沟通成本,你有变化需求,很难及时满足.

作为开发者,要想随时构建项目,那就需要 Travis CI.


### 安装 Jenkins

##### 1. 安装

安装方式一:下载[pkg](https://mirrors.tuna.tsinghua.edu.cn/jenkins/osx/jenkins-2.45.pkg)文件,然后安装即可(此处未必是最新版,请移步[官网](https://jenkins.io/)下载)

安装方式二:终端

``` shell
# 安装自动选择最快源的插件
$ yum install yum-fastestmirror -y  
# 添加 Jenkins 源:
$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://jenkins-ci.org/redhat/jenkins.repo
$ sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
#安装 Jenkins
$ yum install jenkins               
```

##### 2. 启动

``` shell
$ sudo service jenkins start
```

##### 3. 访问方式

``` shell
$ open http://localhost:8080/
```

##### 4. 修改端口

为避免端口冲突,可重配端口:

``` shell
$ vim /etc/sysconfig/jenkins
$ sudo service jenkins restart
```

或者这样修改端口:

``` shell
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort 8443
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
# 如果是 https:
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort 8443
sudo defaults write /Library/Preferences/org.jenkins-ci httpsKeyStore /path/to/your/keystore/file
sudo defaults write /Library/Preferences/org.jenkins-ci httpsKeyStorePassword <keystore password>
# 其中两条命令分别是启动和关闭 Jenkins
```

##### 5. 用户设置

- 打开页面: http://localhost:8080 , 按照提示从`/Users/Shared/Jenkins/Home/secrets/initialAdminPassword`文件中获取密码,然后输入即可. **vim 打开文件或者 cat 显示文件皆可**

- 然后进入插件设置页面,可以选择 Jenkins 推荐的插件,也可自主选择需要安装的插件.(会有一些插件安装失败,没关系,可以后续再安装)

- 安装完成之后,进入创建用户页面,设置用户名密码等

- 用户创建完成,就可以开始创建第一个 Jenkins 任务啦!

主界面如下(已经创建过一个任务):

![](http://img.hb.aicdn.com/0b2dc9f4695effe8bd26d856957165a040f4c3c310b35-pnIrgy_fw658)

##### 6.系统设置

1. 编码设置:

>进入 系统管理->系统设置 界面：

首先设置一下 jenkins 内部shell 执行编码，目的当在jenkins执行shell命令时，有时候会报 utf-8 编码错误。主要是pod install的时候报错。

[33mWARNING: CocoaPods requires your terminal to be using UTF-8 encoding.

设置如下：

![](http://img.hb.aicdn.com/cb6bfa7ef1bc54947df56afd938f66f13617e97b220ed-hPTfFv_fw658)

2.邮件设置:

jenkins Location 设置
主要设置 jenkins 外部访问的 URL 和系统管理员的邮箱地址.用来发送错误报告的邮箱地址：如图
![](http://7xsugd.com2.z0.glb.clouddn.com/runningyoungBlog/images/location.png)

上面只设置了邮箱发送的地址（From地址）,下面设置邮箱的服务器\协议\邮箱\密码.如图:

![](http://7xsugd.com2.z0.glb.clouddn.com/runningyoungBlog/images/email.png)

##### 7. 认证配置

- 基于 用户名+密码 : 将 Gitlab 的用户名,密码添加进来.就可以通过此认证,来访问 Git 的项目库
- 
- 基于 API Token : 将 GIT的 token 添加进来.


### 项目通用配置

-  插件gitlab hook plugin，可以识别gitlab发送过来的提交信息，并触发对应的job
-  Environment Injector Plugin（自定义全局变量）
-  Email Extension Plugin（邮件）



可以先看看下边的官方教程:

[官方指南 教程](https://berlin.gitbooks.io/jenkins/content/job/pei_zhi_bian_yi_can_shu.html)

##### 项目编译参数

设置 debug,test等:

> 打开项目 -> 选择参数化构建过程 -> 点选 Choice 配置如下
> 
> ![](https://berlin.gitbooks.io/jenkins/content/pic/job/jb-02.jpg)

#####  项目构建参数

设置构建参数:

> 打开项目 -> Build with Parameters -> 配置如下
> 
> ![](https://berlin.gitbooks.io/jenkins/content/pic/job/jb-03.jpg)

##### 构建触发器

> 通常手动构建job不是我们想要的，我们想要让其自动构建，这就需要达到一定条件进行触发构建，所有就有了触发器
> 详细的配置,参考[官方文档](https://berlin.gitbooks.io/jenkins/content/job/gou_jian_de_hong_fa_qi.html)


Jenkins 支持多种触发器配置,包括：


- 
- 定期检测代码更新，如有更新则进行构建（Poll SCM） (隔一段时间比较一次源代码如果发生变更，那么就build。否则，不进行build)
- 
- 根据提交进行构建(Build when a change is pushed to GitHub)
-
- 触发远程构建(例如和gitlab的webhooks联动)(当一有对代码push的情况下，就执行构建)

这里以 Gitlab 为例: 有两种方法,这里介绍简单的一种,复杂的请看[官方](https://berlin.gitbooks.io/jenkins/content/job/gou_jian_de_hong_fa_qi.html)

> 1.通过gitlab的webhook与jenkins联动自动触发:需要 jenkins上安装Gitlab Hook Plugin插件.
> 
> 2.勾选:Build when a change is pushed to GitLab. GitLab CI Service URL: http://localhost:8080/project/CI%20GitLab  并保存后边的 URL, 这用于在 gitlab 中的 webhook.
> 
>  保存以上设置,然后:
>  
> 3.进入 gitlab -> 项目 -> 点击左侧的设置 -> webhook -> 粘贴刚才的 URL, 保存,并点击 TEST HOOK... 如果 一切 OK, 页面很快会提示:HOOK SUCCESSFUL.
> 
> 4.有问题,参见[官方](https://berlin.gitbooks.io/jenkins/content/job/gou_jian_de_hong_fa_qi.html)
> 
> ![](https://berlin.gitbooks.io/jenkins/content/pic/job/jb-18.jpg)
> 

- 定期进行构建（Build periodically），(隔一段时间build一次，不管版本库代码是否发生变化，通常不会采用此种方式),语法如下:
```
H(25-30) 18 1-5： 工作日下午6点25到30分之间进行build
H 23 1-5：工作日每晚23:00至23:59之间的某一时刻进行build
H(0-29)/15：前半小时内每隔15分钟进行build（开始时间不确定） 
H/20：每隔20分钟进行build（开始时间不确定）
```


###### Jobs构建方式\编译 配置

Jenkins支持多种编译配置方式,包括：

- Xcode: iOS编译配置（安装Xcode integration插件）
- Invoke Gradle script： Android编译配置(安装Gradle plugin插件)
- Exceute Shell： 脚本方式

对于iOS应用的构建,如果选择Xcode方式构建,需要配置好开发者证书. 推荐使用Exceute Shell方式,简单有效。


##### 构建后的操作

- 向分支push后自动触发jenkins job,上边已经是实现
- jenkins job 成功执行后为当前的commit添加一个标签，方便以后进行线上回滚
- 如果编译失败，就发邮件给开发人员,如果编译成功，就发邮件给测试人员


Artifacts和邮件通知配置等,[参考](https://gold.xitu.io/entry/5792b7320a2b580061ab0e27)


 







### 配置 github 

之所以要配置,是因为 Jenkins 默认不支持 GitHub,需要自行配置(默认支持 CVS\Subversion)

- 1.在 GitHub 中,配置 token (保存此 token 紧接着就要用)

> 流程: github --> setting --> Personal Access Token --> Generate new token

详细配置如下图:

![](http://img.hb.aicdn.com/df55935992514b80c0672f2d3ae55ce4f726f4731bdfe-eVrSec_fw658)

- 2.安装 GitHub Plugin
 
如果此前已经没安装过
> 流程:系统管理-->插件管理-->可选插件-->直接安装 GitHub Plugin
> 
> Jenkins会自动帮你安装plain-credentials 、git 、 credentials 、 github-api

- 3.配置 GitHub Plugin

> 流程:系统管理 --> 系统设置 --> GitHub --> Add GitHub Sever
> 
> Add GitHub Sever 页面 Kind 选择 Secrect text,Secret 贴入上文 GitHub 中配置的 Token,ID 可空白,并加上简单描述即可.

![](http://img.hb.aicdn.com/52b78661c0326778df8ea624bb680b59aa11c884106ee-Gos8u3_fw658)

以上就完成了 GitHub 的配置.

#### 创建freestyle任务

主界面 --> 点击新建 --> 输入任务名称,并选择任务类型,然后确定

![](http://img.hb.aicdn.com/c51fae690f1e3d46ae3b63c432fd275add8808b524354-J7rfU3_fw658)

- 1.基本信息:填入 GitHub 项目主页

![](http://img.hb.aicdn.com/0503e68febf932d309025bba65434d9507d223cde4ae-ECKWwr_fw658)

- 2.项目源码管理:

>填入项目的 git 地址-->点击 Add 添加 git 账户,以便读写该项目-->Branches to build 用于监听分支,**表示监听全部分支-->Local subdirectory for repo 表示项目相对于默认路径的保存路径,之后都会在此路径进行构建执行.

![](http://img.hb.aicdn.com/969a90bcc1d7d12c26164dd7f7ef11b1048a24a3e321-l92EFL_fw658)

- 3.构建触发器/构建环境:

> GitHub hook trigger for GITScm polling 选项:当 GitHub 有版本库更新时触发 Jenkins 进行构建
> Poll SCM 选项:定期检查版本库是否有更新(使用Schedule语法)

![](http://img.hb.aicdn.com/cd592746112fbda093692f630b3b6c8c8e260d4134bac-9nqLy0_fw658)

- 4.构建

![](http://img.hb.aicdn.com/e0ce275c67b89629017b4a706b2b6cc6b283ed5660c6-0fzKor_fw658)

这里指定脚本

``` shell
#!/usr/bin/env zsh --login

rvm use 2.0.0
ruby --version
bundle
bundle --version
bin/pod
/usr/bin/xcodebuild -scheme 'UITEST' -workspace UITEST.xcworkspace -destination "platform=iOS Simulator,name=iPhone Retina (4-inch),OS=latest" -configuration Debug clean build test ONLY_ACTIVE_ARCH=NO
```
- 5\ 构建后的操作
Jenkins默认只能构建前执行shell,不能构建后执行shell.当有该需求时 Jenkins 的 PostBuildScript plugin 插件实现这个功能.

![](http://img.hb.aicdn.com/517fe769dfaed57c8a9e6389d940bbbe1a50bfe83a7c-3uou8P_fw658)


以上便完成了任务的配置.

##### 配置Github仓库的Webhook

> 进入GitHub上指定的项目 --> setting --> WebHooks&Services --> add webhook --> 输入刚刚部署jenkins的服务器的IP

这样,每次 push 都会触发一次build.

然而,我的 Jenkins 是本地的,在 GitHub 并不能直接访问到我的电脑,so,设置是无效的....


### 配置 GitLab

**集成目的: Gitlab 有新的推送,就会触发 Jenkins 构建.构建状态反馈给 Gitlab**

##### 1. 安装插件

> [Gitlab官方教程](https://docs.gitlab.com/ee/integration/jenkins.html)
> [Gitlab 官方教程2](https://docs.gitlab.com/ee/integration/jenkins.html)

- 1). 安装 Gitlab 插件 

> 系统管理 -> 管理插件 -> 可选插件 :中选择GitLab Plugin 和 Gitlab Hook Plugin 这两项，然后安装.

- 2). 安装 Xcode 插件

> 同上,安装 Xcode integration

- 3). 安装签名证书管理插件
OS打包内测版时,需要发布证书及相关签名文件,因此这两个插件对于管理iOS证书非常方便.

> 同上,安装 Credentials Plugin 和 Keychains and Provisioning Profiles Management 

##### 2. 配置 API Token

> 1. Gitlabe: Copy the private API token from [ Profile Settings -> Account ]
> 
> 2. Jenkins: Go to [ Manage Jenkins -> Configure System ] -> [GitLab] 粘贴 Gitlab 主机地址,以及 API Token
> 
> 3. 可能2步骤没有 API Token 选项,因为都已经集成到 Credentials 中了,所以,要进入 Credentials 中,然后 添加新的凭证:
> 
> 首页点击 Credentials -> 点击 System -> 点击底部的 Glopbal credentials(unrestricted) -> 跳入新的页面,点击 Add Credentials -> Kind 选择 [GitLab API Token],然后粘贴上述的 token ,要注意的是, Description 需要写清楚,不然容易弄混淆(GitHub 也添加了 Secret text, 容易混)
>
> 以上前提是有安装插件:Plain Credentiles Plugin ,以及相关插件. 否则不会有 gitlab API token 这个选项的.

##### 3. 配置 Gitlab 账号密码


在系统配置里,添加 类型: Username with password -> 配置账号密码即可.


##### 4. 配置 SSH

1.直接获取:

``` shell
$ ls ~/.ssh
$ cat id_rsa # 显示
$ pbcopy < ~/.ssh/id_rsa #复制
# 然后 粘贴到 Jenkins 中,并输入最初始的密码即可.如下图
```

![](http://img.hb.aicdn.com/9756ed06006b5d816ac3e4fdc9891249fd7879f116026-K0Lsxk_fw658)

2.创建新的 SSH

``` shell
$ ssh-keygen -t rsa -C “Your email” 
# 私钥id_rsa:本机添加秘钥到秘钥管理器:
$ eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_rsa
# 私钥copy添加进 Jenkins（private key选项）
$ pbcopy < ~/.ssh/id_rsa
# 公钥id_rsa.pub:复制id_rsa.pub里面的公钥添加到Gitlab上  
$ pbcopy < ~/.ssh/id_rsa.pub

```

SSH 添加路径为:

> 系统管理 -> 系统设置 -> Gitlab 栏 点击 Add 添加,或者使用如下地址:
> 
> http://localhost:8080/credentials/store/system/domain/_/


#### 创建freestyle任务

主界面 --> 点击新建 --> 输入任务名称,并选择 freestyle 任务类型,然后确定

- 源码管理:

>> 输入项目 git 协议地址(git@your.gitlab.server:gitlab_group/gitlab_project.git) 
>> --> 证书类型选择:相应的 SSH .
>> -->在高级设置中，将名称设置为origin,将Refspec设置为 +refs/heads/*:refs/remotes/origin/* +refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*

- 
- 构建触发器:Build when a change is pushed to GitLab. 注意,这里的地址,粘贴在 Gitlab 上 的 WebHook 中
- 

- 更加详细的任务配置,参考插件[官方文档](https://github.com/jenkinsci/gitlab-plugin#using-it-with-a-job)





### Xcode Builder

###### Xcode Builder 设置

> 系统管理–>系统设置->Xcode Builder
> 默认 path 不变 -> 点击 Add 需填写的内容：
>
>![](http://img.blog.csdn.net/20160216141930768)



>>Keychain Name：iPhone Distribution: *（dis证书常用名）
>>
>>Keychain path：${HOME}/Library/Keychains/login.keychain（dis证书路径）
>>
>>Keychain password：*
>>
>>Add to keychain search path after build：Yes
>>
>>Default keychain:iPhone Distribution: *

- 1. General build settings: 

> ++ 填写Target 名字
> 
> ++ Clean before build? YES
> 
> ++ Pack application and build .ipa? YES
> 
> ++ .ipa filename pattern IPA 路径:AD_Demo_${SHORT_VERSION}(固定写法:targetName_${SHORT_VERSION})
> 
> ++ Output directory IPA输出位置 ${WORKSPACE}/build/${BUILD_NUMBER}/
> 

- 2.Code signing & OS X keychain options: 

在系统Xcode Builder（钥匙串设置）已配置,上边已配置
- 3.Advanced Xcode build options: 
> ++ “Clean test reports?”  YES
> 
> ++Build output directory：${WORKSPACE}/build/${BUILD_NUMBER}/
> 

![](http://img.blog.csdn.net/20160218095046760)
![](http://img.blog.csdn.net/20160218095059932)
![](http://img.blog.csdn.net/20160218095117084)
- 4. Versioning 构建后的设置 :SHELL 脚本

![](http://img.blog.csdn.net/20160218100247953)

1).点击增加 Shell 脚本:

``` shell 
"${WORKSPACE}/build/${BUILD_NUMBER}"

for file in "*.ipa"
do
    PLIST_NAME=`echo $file`
done

PLIST_NAME=${PLIST_NAME%.*}

cd "${WORKSPACE}/build"
echo "PLIST_NAME=$PLIST_NAME" > jenkinsUserGlobal.properties

cat << EOF > ${WORKSPACE}/build/${BUILD_NUMBER}/$PLIST_NAME.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>items</key>
        <array>
                <dict>
                        <key>assets</key>
                        <array>
                                <dict>
                                        <key>kind</key>
                                        <string>software-package</string>
                                        <key>url</key>
                                        <string>https://127.0.0.1/$PLIST_NAME.ipa</string>
                                </dict>
                        </array>
                        <key>metadata</key>
                        <dict>
                                <key>bundle-identifier</key>
                                <string>com.3code.ADDemo</string>
                                <key>bundle-version</key>
                                <string>1</string>
                                <key>kind</key>
                                <string>software</string>
                                <key>title</key>
                                <string>ADDemo</string>
                        </dict>
                </dict>
        </array>
</dict>
</plist>
EOF
```

2). 点击增加 Inject environment variables

并设置:
> 需填写的内容：(将在Execute shell中生成的全局变量文件地址放入 Properties File Path中)
Properties File Path：${WORKSPACE}/build/jenkinsUserGlobal.properties

![](http://img.blog.csdn.net/20160218100610924)

- 5.构建后的操作: 设置邮件

![](http://img.blog.csdn.net/20160218100726636)
触发器：构建成功、失败触发(扩展一下：失败点“高级”设置只发送给自己) 
![](http://img.blog.csdn.net/20160218100837965)

>需填写的内容：
>
> Project Recipient List：邮件接收人（多个时用”,“分割）
>
> Project Reply-To List：$DEFAULT_REPLYTO
> 
> Content Type：选择”HTML(text/html)“
> 
> Default Subject：标题: **${PROJECT_NAME}构建通知:第${BUILD_NUMBER}次持续集成${PLIST_NAME}构建${BUILD_STATUS}**
> 
> Default Content:（将ipa全局变量传递到邮件生成下载链接和链接二维码）,如下:

``` xml
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
</head>

<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
    ffset="0">
    <table width="95%" cellpadding="0" cellspacing="0"
        style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
        <tr>
            <td><h2>
                    <font color="#0000FF">构建结果 - ${BUILD_STATUS}</font>
                </h2></td>
        </tr>
        <tr>
                <td><h2>
                    <font color="#FF0000">App下载链接:<a href="itms-services://?action=download-manifest&url=https://******/${PLIST_NAME}.plist">itms-services://?action=download-manifest&url=https://127.0.0.1/${PLIST_NAME}.plist</a></font>
                </h2></td>
        </tr>
        <tr>
               <td><h2>
                      <font color="#FF0000">二维码图片:</font>
               </h2></td>
        </tr>
        <tr>
                <td>
                    <img src="http://qr.liantu.com/api.php?text=itms-services://?action=download-manifest%26url=https://******/${PLIST_NAME}.plist" height="300" width="300">
                </td>
        </tr>
        <tr>
            <td><br />
            <b><font color="#0B610B">构建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>项目名称&nbsp;：&nbsp;${PROJECT_NAME}</li>
                    <li>构建编号&nbsp;：&nbsp;第${BUILD_NUMBER}次构建</li>
                    <li>SVN&nbsp;版本：&nbsp;${SVN_REVISION}</li>
                    <li>触发原因：&nbsp;${CAUSE}</li>
                    <li>构建日志：&nbsp;<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                    <li>构建&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                    <li>工作目录&nbsp;：&nbsp;<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                    <li>项目&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><b><font color="#0B610B">Changes Since Last
                        Successful Build:</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>历史变更记录 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
                </ul> ${CHANGES_SINCE_LAST_SUCCESS,reverse=true, format="Changes for Build #%n:<br />%c<br />",showPaths=true,changesFormat="<pre>[%a]<br />%m</pre>",pathFormat="&nbsp;&nbsp;&nbsp;&nbsp;%p"}
            </td>
        </tr>
        <tr>
            <td><b>Failed Test Results</b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td><pre
                    style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
                <br /></td>
        </tr>
        <tr>
            <td><b><font color="#0B610B">构建日志 (最后 100行):</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td><textarea cols="80" rows="30" readonly="readonly"
                    style="font-family: Courier New">${BUILD_LOG, maxLines=100}</textarea>
            </td>
        </tr>
    </table>
</body>
</html>
```

邮件截图

![](http://img.blog.csdn.net/20160218103409264)
![](http://img.blog.csdn.net/20160218103358248)

### 多 Target 扩展


### Jenkins Shell 脚本

在 Jenkins 的任务里面,在构建\构建后,都可选择 Execute shell ,作为构建前或者构建后要执行的 shell 脚本.

python\perl\ruby 等脚本其实也是 shell 脚本,因此这里的 Shell 可以扩展为 python\perl\ruby
等。


常见脚本如:

1\此脚本:在jenkins构建完成后,远程copy配置文件到项目中,并重启tomcat

``` shell
#!/bin/sh
#datetime: 2017-2-7 17:00
#author:3code
#desc:此脚本用来在jenkins构建完成后，远程copy配置文件到项目中，并重启tomcat

#copy API工程的配置文件
cp -r /data/apache-tomcat-7.0.56/build/API/*  /data/apache-tomcat-7.0.56/webapps/API/WEB-INF/classes

#重启tomcat
bash /data/apache-tomcat-7.0.56/bin/shutdown.sh

sleep 30s

bash /data/apache-tomcat-7.0.56/bin/startup.sh
```



**Tips:**
- 对于刚接触Jenkins的同学可能有很多需要配置的地方不知道要怎么填，这时可以点击每一个填空后的？，就会弹出详细的提示。
- 官方的wiki是很好的入门教程https://wiki.jenkins-ci.org/display/JENKINS/Home


### 其他

- Jenkins 的界面是可以自定义的,[参考此处](https://wiki.jenkins-ci.org/display/JENKINS/Simple+Theme+Plugin)
- [优秀:Jenkins + Gitlab + 蒲公英 + 邮件通知](https://runningyoung.github.io/2016/04/01/2016-04-05-jenkins2/)
- [优秀:jenkins+xcode+svn+七牛](http://blog.csdn.net/fengshi_sh/article/details/50669754)



[构建iOS持续集成平台（三）——CI服务器与自动化部署](http://www.itgo.me/a/x9151986573431569483)
[fir.im weekly - 「 持续集成 」实践教程合集](http://blog.fir.im/fir_im_weekly160505/)
[谈谈持续集成，持续交付，持续部署之间的区别](http://blog.flow.ci/cicd_difference/)
[使用Travis CI自动部署React Native项目（iOS篇）](http://blog.caoyue.me/post/react-native-travis-ci-ios)
[构建iOS持续集成平台（三）——CI服务器与自动化部署](http://www.infoq.com/cn/articles/build-ios-continuous-integration-platform-part3)
...
[为 iOS 建立 Travis CI](https://objccn.io/issue-6-5/)
[iOS 持续集成 --Travis CI + Fir.im 自动编译发布](https://gold.xitu.io/entry/57a982d10a2b58005869d8dc)
[iOS 持续集成--Travis CI + Fir.im自动编译发布](http://yeziahehe.com/2016/08/07/use_travis_ci_for_ios_project/)
[为 iOS 建立 Travis CI](https://objccn.io/issue-6-5/)
[在Docker Hub上配置自动构建](https://docs.docker.com/docker-hub/builds/)
[利用pod trunk发布程序](http://www.jianshu.com/p/19c2b13e81fa)
[使用GitLab来实现IOS项目的持续集成CI](http://www.55118885.com/w/166141242.html)
[Jenkins + GitHub + fir-cli 一行命令从源码到 fir.im](http://blog.fir.im/jenkinsgithubfir_cli-xing-ming-ling-cong-yuan-ma-dao-fir-im/)
[用持续集成工具Travis进行构建和部署](http://www.2cto.com/kf/201411/356826.html)
#### fir.im
[使用 fir CLI插件上传 Swift 应用至fir.im](http://www.jianshu.com/p/f17caec72f0b)

#### Travis
相关的文章:[Travis](https://blog.jamespan.me/2015/11/01/ci-your-hexo-blog)





>[使用 TestFlight 进行 App Beta 版测试](http://ios.jobbole.com/87803/)
>[使用TestFlight进行App构建版本测试](http://www.jianshu.com/p/27545c2d4d8b)
>[onevcat testflight](https://onevcat.com/2012/01/testflight/)
持续部署是指当交付的代码通过评审之后，自动部署到生产环境中。持续部署是持续交付的最高阶段。这意味着，所有通过了一系列的自动化测试的改动都将自动部署到生产环境。它也可以被称为“Continuous Release”。


## 自动发布

- Fastlane 是一组工具套件,旨在实现iOS应用发布流程的自动化，并且提供一个运行良好的持续部署流程.
- 使用场景: push 时执行单元测试,集成测试;创建屏幕截图发给客户;构建分发 Beta 版本;构建并分发到 appstore;
- 支持与Jenkins和CocoaPods，xctools等其他第三方工具的集成
- 博客教程:http://www.infoq.com/cn/news/2015/01/fastlane-ios-continuous-deploy
- 官方教程:https://docs.fastlane.tools/getting-started/ios/screenshots/
 






