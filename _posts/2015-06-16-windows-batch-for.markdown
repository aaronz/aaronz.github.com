---
layout: post
title:  "Windows Batch"
date:   2015-06-16 00:00:00
categories: Batch
---

最近总是要通过一系列的命令抓些数据进行本地测试，数据的抓取过程繁琐而重复，于是想通过一个简单的脚本代替。脚本语言多种多言，python, powershell, vbs, 这次尝试了下古老的windows batch。
于是发现windows batch里面有很多有趣的命令和用法。

![batch](/assets/images/posts/batch.jpeg)

<!--more-->

# 简单示例

下面是一个简单的batch方法，他通过读取系统的时间定义了若干个变量，如%YYYYMMDD%、%HHMMSSSS%、%HHMMSS%和%HHMM%。

{% highlight text %}
::===========================================================================
:GetDateTime
:: Tokenise date into DD MM and YY independent of locale
:: NEWSGROUP: microsoft.public.win2000.cmdprompt.admin
:: SUBJECT  : How can I to get the current month in commandline?
:: WHEN/WHO : Mar 14 2001, 9:05 pm  post by Michael (maj0)
::
:: DATE output (when prompting for new date) looks like:
::      The current date is: Sat 05/07/2014
::      Enter the new date: (dd-mm-yy)
::
:: The first loop reads the DATE from the environment variable and splits it up
:: The second loop reads 2nd line of the date output and splits up "(dd-mm-yy)"
:: It then sets up the "DD", "MM" and "YY" environment variables.
::===========================================================================
    ::--- Get the date ------------------------------------------------------
    for /f "tokens=2-4 delims=.:/-, " %%i in ("%date%") do (
        for /f "tokens=2-4 delims=/-,() skip=1" %%l in ('echo.^|date') do (
          set %%l=%%i
          set %%m=%%j
          set %%n=%%k
        )
    )
    set YYYYMMDD=%YY%-%MM%-%DD%
 
    ::--- Get the day of the week -------------------------------------------
    for /f "tokens=1 delims= " %%W in ("%date%") do set Day3=%%W
 
    ::--- Get some time formats ---------------------------------------------
    set HHMMSSSS=%TIME: =0%
    set   HHMMSS=%HHMMSSSS:~0,8%
    set     HHMM=%HHMMSSSS:~0,5%
    goto :EOF

{% endhighlight %}

# 示例分析

在这个简单的示例中，体现了windows batch的几个基本技巧。 

- 变量定义通过 set variablename=variablevalue 
- 字符串处理通过 for 语句对字符串进行分拆和组合 
- 调用方法可以通过 call :GetDateTime 

# for 命令的使用方法

for 命令和编程语言中常见的 for 循环命令有很大差别。先看看for支持的常用选项，

- delims 用作字符串处理中的分隔符，支持多个
- skip 用作跳过字符串的前几行
- tokens 用作指定被处理的字符串坐标
 
举一个简单例子 

weather.txt文件内容如下

{% highlight text %}

January,Snowy,02 
February,Rainy,15 
March,Sunny,25 
 
{% endhighlight %}

for /f "tokens=1,3 delims=," %%G in (weather.txt) do @echo %%G %%H

- 因为指定分隔符为逗号，字符串被分拆成January Snowy 和 02 
- 同时因为制订了tokens=1,3，结果将会掠过坐标为2的snowy 
- 赋值变量%%G作为起始变量，%%H作为第二个变量，最终输出结果如下 

{% highlight text %}
January 02 
Febuary 15 
March 25 
{% endhighlight %}




