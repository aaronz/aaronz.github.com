---
layout: post
title:  "如何在windows phone internet explorer中调试网页应用"
date:   2014-10-22 00:00:00
categories: Debugging
---

最近遇到了一个棘手的Windows Phone 8.1中IE的问题，试用了各种调试方式，从工具角度来讲Visual Studio对windows phone调试支持已经相当不错了，这篇文章来总结下如何在windows phone internet explorer中调试网页应用的。

![wp-ie](/assets/images/posts/internetexplorer-windows.jpg)

<!--more-->

桌面浏览器对网页应用调试的支持已经非常丰富了，IE/Chrome/FF的F12开发者工具对调试功能的支持仍然在不停地增加，然而现在很多的网页应用不仅仅需要运行在桌面浏览器上，也需要运行于各种手机浏览器，相对来说手机浏览器的调试支持少的可怜，如果一个问题只在手机浏览器上出现，那么你就要消耗更多的体力和脑力与小屏幕们搏斗了一番了。本文主要介绍Windows Phone上面的Internet Explorer调试网页应用的方法。

# 工具安装

调试过程主要用到的工具有下面几种，

1. [Fiddler4](http://www.telerik.com/download/fiddler) Fiddler主要负责抓包，如果问题发生环境与调试环境隔离，还可以通过fiddler来反复重演抓包进行调试。
2. [Visual Studio](https://dev.windows.com/en-us/develop/download-phone-sdk) VS目前支持虚拟机中的Internet Explorer程序调试，后面详细介绍方法。
3. [Windows Phone SDK](https://dev.windows.com/en-us/develop/download-phone-sdk) SDK中可以选择符合问题出现的WP版本的虚拟机。
4. [桌面版Internet Explorer 11](http://www.microsoft.com/en-us/download/internet-explorer-11-details.aspx) 桌面版IE可以模拟WP IE，但是并非完全模拟，例如触碰事件在桌面版IE是不支持的。

# Visual Studio 动态调试模拟器

Visual Studio 2013与Windows Phone SDK安装好后，在VS的Tool菜单中可以找到Windows Phone Developer Power Tools，打开之后会启动下面这样一个工具，它可以帮你启动各种型号版本的Windows Phone模拟器。

![windows phone developer power tools](/assets/images/posts/windows-phone-developer-power-tools.png)

在设备菜单中选择好需要的型号之后点击Connect按钮，就会启动一个对应版本的模拟器如下。

![windows phone emulator](/assets/images/posts/wp-emulator.png)

在模拟器中你可进行各种各样的操作来重现问题。接下来我们就来说调试。
通过Visual Studio的debug菜单，我们可以选择Debug other target来选择模拟器中的Internet Explorer作为调试目标。

![vs choose debug target](/assets/images/posts/vs-wp-debug-ie-target.png)

点击该按钮之后就会弹出下面对话框来指定要调试的页面。

![choose debug page](/assets/images/posts/choose-ie-setting.png)

然后就可以开始调试了，这时VS会自动attach到模拟器中的IE进程，然后通过VS就可以进行模拟器IE中的DOM inspection, 各种CSS/Events/Properties都一目了然，也可以选择页面加载的脚本设置断点，查看调用栈和变量情况。

![live debug wp ie](/assets/images/posts/live-debug-wp-ie.png)

# 桌面版IE模拟Windows Phone IE

桌面版的IE同样也可模拟Windows Phone版的IE，目前支持WP7-IE9和WP8-IE10，后续应该会添加更多的版本支持。通过桌面IE模拟器来进行调试我们就可以使用强大的F12开发者工具，进行layout/script/network/memory等各方面的调试。

![desktop IE silumator](/assets/images/posts/desktop-ie-simulate-wp-ie.png)



