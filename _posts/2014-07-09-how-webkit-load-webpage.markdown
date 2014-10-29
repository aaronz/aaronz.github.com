---
layout: post
title:  "WebKit页面加载"
date:   2014-07-09 00:00:00
categories: Mechanism
---

这篇文章翻译自WebKit博客中的一篇介绍WebKit加载原理的文章。原文链接如下，

[http://www.browserwork.com/mechanism/webcore-renderring-floats/](http://www.browserwork.com/mechanism/webcore-renderring-floats/)

WebKit在渲染页面之前首先要将网页内容及其依赖的内容从网络上下载下来。加载这些页面内容的工作实际上涉及到了多个层次上的工作。在这篇文章中，将会重点描述WebCore在内容加载过程中所起的作用。

![loading](/assets/images/posts/load70.jpeg)

<!--more-->

WebKit包含两个加载通道，其一用来加载主页面，其二用来加载依赖文件(例如图片或脚本等)。下图简要概括了两个通道中的主要对象。

![Webkit load webpage](/assets/images/posts/webkit-load-webpage.jpg)

# Frames加载

[FrameLoader](http://trac.webkit.org/browser/trunk/Source/WebCore/loader/FrameLoader.cpp)负责加载文档并将其转化为Frames。当点击一个链接，FrameLoader会创建一个[DocumentLoader](http://trac.webkit.org/browser/trunk/Source/WebCore/loader/DocumentLoader.cpp)并将其置于policy状态，在该状态下它等待WebKit决定如何处理这次加载。通常情况下，客户端会让FrameLoader把加载作为一次导航(而不是一次阻塞加载)。

当客户端让FrameLoader将本次加载作为一次正常的导航后，FrameLoader会将DocumentLoader置于provisional状态，从而发起网络请求来确定网络请求是下载文件还是一份文档。

DocumentLoader创建[MainResourceLoader](http://trac.webkit.org/browser/trunk/Source/WebCore/loader/ResourceLoader.cpp)来通过ResourceHandle和系统的网络接口进行交互。将DocumentLoader和MainResourceLoader加以区分主要有两个原因，其一是MainResourceLoader使DocumentLoader与[ResourceHandle](http://trac.webkit.org/browser/trunk/Source/WebCore/platform/network/ResourceHandle.h)的回调方法相互隔离；其二是MainResourceLoader的可以与DocumentLoader的生命周期(DocumentLoader的生命周期与Document一致)解耦。

当浏览器加载了足够的信息来都来呈现网页后，FrameLoader将DocumentLoader的状态转换为committed状态，从而开启Frame的文档呈现过程。

# 子资源加载

然而加载网页不仅仅需要HTML页面，同时也需要加载图片，脚本及其他HTML页面引用的内容。DocLoader会负责加载这些子类资源。(注意DocLoader并非DocumentLoader，他们名字相似，但是职责不同。)

通过加载图片的过程作为示例。为了加载一个图片，DocLoader首先会查询缓存对象中是否已经有一份缓存的拷贝(CachedImage)。如果图片已经在缓存中，DocLoader可以马上返回这个图片。为了保持高效，缓存中通常保存解码后的图片，因此WebKit从缓存中读取图片是不需要重复解码工作。

如果图片不在Cache中，Cache会创建一个CachedImage对象来代表这个图片。CacheImage对象会通知Loader对象来启动一个网络请求，Loader的工作就是创建一个[SubresourceLoader](http://trac.webkit.org/browser/trunk/Source/WebCore/loader/SubresourceLoader.cpp)来做和MainResourceLoader类似的工作，通过ResourceHandle与系统网络接口交互来下载文件。

# 可改进之处

WebKit的加载通道有很多需要改进的地方。FrameLoader被设计的过于负责，其真正的职责应该定位在简单的加载frame，例如FrameLoader包含多个Load方法，容易让人疑惑，另外它还负责创建窗口，这件事和加载frame没多大关系。另外加载通道的各个阶段耦合的过于紧密，有些低层次的对象在于高层次对象进行交互。例如MainResourceLoader会调用FrameLoader来传递从网络层收到的数据，而不是将其传递给DocumentLoader。

如果了解了文章开始的层次涉及图，会注意到缓存只被应用于子类资源。主页面加载并没有从WebKit的内存缓存中获益。如果可以将两个加载通道统一，也许可以使主页面的加载获得更多的性能提升。希望能够在日后不断进行资源加载的优化使其越来越快。


