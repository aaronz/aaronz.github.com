---
layout: post
title:  "如何用VS2013和Windows8.1编译webkit"
date:   2014-07-01 00:00:00
categories: How-to
---

如果想要了解WebKit难免要将整个项目源代码下载下来重新编译，然后亲手调试一番。WebKit在Windows平台上可以很容易的完成编译。下面是在Windows 8.1上使用VS2013编译WebKit的步骤，想同样了解WebKit的朋友可以参考尝试一下。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# 安装软件

1. 下载安装[Visual Studio 2013](http://www.microsoft.com/zh-cn/download/details.aspx?id=40763)或者[Visual Studio 2013 express](http://www.microsoft.com/zh-cn/download/details.aspx?id=40748)版本，默认安装即可。
2. 安装Cygwin。Cygwin提供了WebKit源代码编译所需要的一组工具。WebKit站点提供了个一包含所需工具的Cygwin安装包。通过[这里](http://svn.webkit.org/repository/webkit/trunk/Tools/CygwinDownloader/cygwin-downloader.zip)下载。
    + 将下载的压缩包解压
    + 运行 cygwin-downloader.exe，它会开始下载各种所需的安装包。
    + 当所有的包下载完毕后，cygwin安装程序会自动启动。
    + 选择Install from local directory，选择下一步直至安装完毕。
    + 默认情况下Cygwin会安装pythong 2.7, 但是编译webkit需要2.6.8, 可以启动cygwin安装，选择install from internet来选择安装python 2.6.8版本。
    + 另外需要通过Cygwin安装依次安装gcc->Devel->"gcc-g++: GNU Compiler Collection(C++)", gdb->Devel->"gdb: The GNU Debugger"
    + 打开c:\cygwin\etc\profile, 更改第32行为：PATH="/bin:${PATH}" ，通过在行首添加#注释掉44-50行TMP variable相关内容。
3. 下载安装[June 2010 DirectX SDK](http://www.microsoft.com/en-us/download/details.aspx?id=6812)。
4. 安装[QuickTime SDK](http://developer.apple.com/quicktime/download/)和quicktime。

# 源代码

1. 下载WebKit源代码，最快捷的方法是下载一个[WebKit nightly build的snapshot](http://nightly.webkit.org/files/WebKit-SVN-source.tar.bz2)。
2. 通过cygwin命令下使用下面命令解压源代码压缩包。
{% highlight text %}
tar jxvf WebKit-SVN-source.tar.bz2
cd webkit
{% endhighlight %}
3. 下载[WebKit Support Library](http://developer.apple.com/opensource/internet/webkit_sptlib_agree.html)到源代码根目录c:\webkit\。这个支持包需要命名为WebKitSupportLibrary.zip，不用解压。
4. 运行Tool/Scripts/update-webkit来更新代码树。

# 编译

1. 启动Cygwin命令行，转向WebKit/Tools/Scripts/，运行build-webkit --debug编译代码。如果希望了解更多支持的参数，可以通过build-webkit --help。
2. 直接打开webkit\Source\WebKit\WebKit.vcxproj\webkit.sln，通过Visual Studio编译。
3. 编译后到webkit\WebKitBuild\Debug\bin32启动WinLauncher.exe。

# 常见错误

1. 如果update-webkit脚本运行报错"CURL: ssl version is unsupported"，可以修改Tools/Scripts/update-webkit-dependency 85行和116行，将sslv3改为tlsv1。
2. [http://trac.webkit.org/wiki/BuildingOnWindows#CommonBuildErrors](http://trac.webkit.org/wiki/BuildingOnWindows#CommonBuildErrors) 