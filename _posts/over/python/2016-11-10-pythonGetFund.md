---
layout: post
title:  "python脚本 - 开机自动爬基金数据"
categories: python  
tags: python
author: 3code
---

* content
{:toc}

&ensp;&ensp;&ensp;&ensp;话说玩基金也有很长一段时间了，但是感觉各种基金app里面的数据展示不全，
不少app都只展示近几年的数据，基金成立之处数据没有，虽然网站上有
但是网站上也只展示在单个基金里，多基金对比就没
成立之初的数据，反正各种不爽，所以萌生直接爬取的想法，刚好可以练练python。

&ensp;&ensp;&ensp;&ensp;我是从15年底就开始运行这个脚本，目前已经有一份数据，正准备借助MATLAB的simulink模块分析。


&ensp;&ensp;&ensp;&ensp;最开始直接放在txt里，后来想存数据库，但是pyCharm上各种安装不上mysql
，不论是pyCharm还是pip还是直接托安装包，
后来改用excel，数据直接写入excel中。

&ensp;&ensp;&ensp;&ensp;截止发文为止，数据已经迁至数据库，相关操作，后续再开一个博文介绍。

&ensp;&ensp;&ensp;&ensp;下边是基于OSX10.11系统，开机自动爬取数据，并存入指定excel，
代码在github[传送门](https://github.com/mythkiven/getFund)

实际展示：

控制台打印：

![](https://ooo.0o0.ooo/2016/11/11/5825712e47466.png)

写入txt：

![](https://ooo.0o0.ooo/2016/11/11/5825719ba664e.png)

写入xls：

![](https://ooo.0o0.ooo/2016/11/11/58257180958e8.png)


## 1、python爬取原理及代码

**原理:获取html – 使用正则匹配 – 解析 – 存储**

##### 1.1 引入需要的库

```
import xlrd
import xlwt
import time, datetime
from xlutils.copy import copy
from xlrd import open_workbook
from xlwt import easyxf
```

##### 1.2 获取html


```
 def getSource(self, url):
        html = requests.get(url)
        return html.text

```

##### 1.3 获取内容

这里匹配内容使用的是 re模块，re是Python自带的正则表达式库文件，为字符串的匹配筛选提供了极大的便利。下文使用re.search，这个模块里还有个相似的函数 re.match()
。区别在于:

match只从字串的开始位置进行匹配，如果失败，它就此放弃；

search则会完全遍历整个字串中所有可能的位置，直到成功地找到一个匹配，或者搜索完字串，以失败告终。


```
def getMainText(self, source):
        everyclass = re.search(
            '(<table width="950" border="0" align="center" cellpadding="0" cellspacing="0" class="table02" id="fundlist" data-lt="sypm">.*?</table>)',
            source, re.S).group(1)
        return everyclass
```

##### 1.4 获取每条信息

将自己需要的信息单独抽取出来：

```
def getInfo(self, eachclass):
        info = {}
        info["num"] = re.search('<td width="3%" align="center">(.*?)</td>', eachclass, re.S).group(1)
        info["id"] = re.search('target="_blank">(.*?)</a></td>', eachclass, re.S).group(1)
        info['name'] = re.search('target="_blank" class="blue ellipsis" title="(.*?)">', eachclass, re.S).group(1)
        info['time'] = re.search('<td width="7%" align="center">(.*?)</td>', eachclass, re.S).group(1)
        info["price1"] = re.findall('<td width="6%" align="center">(.*?)</td>', eachclass, re.S)[1]
        info["price2"] = re.findall('<td width="6%" align="center">(.*?)</td>', eachclass, re.S)[2]

        return info
```

##### 1.5 写入文件

创建txt：

```
f = open("/Users/www_3code_info/fund/" + timePrevious + ".txt", 'a+')
        for each in classinfo:
            f.writelines('xuhao:' + each['num'] + '\n')
            f.writelines('daima:' + each['id'] + '\n')
            f.writelines('name:' + each['name'] + '\n')
            f.writelines('time:' + each['time'] + '\n')
            f.writelines('price1:' + each['price1'] + '\n')
            f.writelines('price2:' + each['price2'] + '\n')
            f.writelines('url:http://www.jjmmw.com/fund/' + each['id'] + '/' + '\n\n')
        f.close()
```

打开xls,开始写入内容

```
# 打开文件
        rb = open_workbook(path, formatting_info=True)
        r_sheet = rb.sheet_by_index(0)
        wb = copy(rb)
        sheet = wb.get_sheet(0)
        j = r_sheet.ncols
        i = 1
        timeShould = timePrevious
        timeQ = timePrevious
        tListB = []
        tListS = []
        print sheet

        fir = ['ID', 'Time', 'NowPrice', 'PlusPrice', 'WriteTime']
#写入列信息
        for each in classinfo:
            sheet.write(i, j + 0, each['id'])
            sheet.write(i, j + 1, each['time'])
            sheet.write(i, j + 2, each['price1'])
            sheet.write(i, j + 3, each['price2'])
            i += 1
            timeQ = each['time']
            if timeQ > timeShould:
                tListB.append(each['id']+' '+timeQ)
            elif timeQ < timeShould:
                tListS.append(each['id']+' '+timeQ)

        for k in range(j, j + 5):
            sheet.write(0, k, fir[k - j])
            print k
            if k == j + 4:
                sheet.write(1, k, timeNow.decode('utf-8'))
                sheet.write(2, k, 'UNupdate:' + str(len(tListS)))

```

##### 1.6 防止数据重复写入

按时间判断是否需要写入数据

```
def read_excel():
    rb = open_workbook(path, formatting_info=True)
    r_sheet = rb.sheet_by_index(0)
    # 遍历第2行所有列,是否有时间数据c
    print "\n已经爬取的列数:" + str(r_sheet.ncols) + "\n"
    for c in range(0, r_sheet.ncols):
        age_nov = r_sheet.cell(1, c).value
        print age_nov
        if ((c + 1) % 5 == 0) and (c != 0):
            print "\n"
        if age_nov == timeNow:
            return 1
    else:
        return 0

```

##### 1.7 按日期爬取数据

基金一般都是晚上更新当天数据，因为开机早上获取，所以这里设置爬取日期是:周2-6。周日,周一不更新

注意weekday() 返回的是0-6是星期一到星期日

```
def getTime():
    timeT = datetime.date.today()
    zhouji = timeT.weekday()
    print zhouji
    if zhouji >= 1 and zhouji <= 5:
        return timeToday  
    else:
        print "今天是周日或者周一,不获取信息。周六获取周五,周二获取周一的信息"
        return 0

```


## 2、开机自动爬取数据的脚本

开机启动相关的可参考我的另一篇文章：[OSX环境变量的设置](www.3code.info/2016/04/30/deployEnvironmentVariable/)


&ensp;&ensp;&ensp;&ensp;在linux下执行定期任务可以使用crontab，mac os上不推荐使用。mac os上推荐做法是采用plist脚本，plist脚本可以设置执行的动作，时间间隔等其他一些信息。使用plist脚本原则上时间间隔可以为一秒。

plist脚本存放路径为:

/Library/LaunchDaemons:只要系统启动了，哪怕用户不登陆系统也会被执行。

或/Library/LaunchAgents:当用户登陆系统后才会被执行。




&ensp;&ensp;&ensp;&ensp;可以通过两种方式来设置脚本的执行时间。一个是使用StartInterval，它指定脚本每间隔多长时间（单位：秒）执行一次；另外一个使用 StartCalendarInterval，它可以指定脚本在多少分钟、小时、天、星期几、月时间上执行，类似如crontab的中的设置。

一个简单例子如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.3code.test.plist</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/3code/plist-test.sh</string>
    </array>
    <key>KeepAlive</key>
    <false/>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>60</integer>
</dict>
</plist>
```

其中key是plist脚本定义的属性，紧跟着的下一行是该属性对应的值。上述脚本是每间隔60秒执行一次/Users/3code/plist-test.sh这个shell脚本，也可以使用StartCalendarInterval来替换StartInterval达到同样的效 果，例如：

<key>StartCalendarInterval</key>
<dict>
  <key>Minute</key>
  <integer>0</integer>
</dict>
上述设置的意思为每天的每个小时的第0分钟执行，也即使每60秒执行一次。

plist脚本中定义的属性以及具体的含义，可以参看苹果官方网站的说明。










 