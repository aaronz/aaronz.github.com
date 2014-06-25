---
layout: post
title:  "WebKit渲染 - blocks 和 inlines"
date:   2014-06-24 00:00:00
categories: Mechanism
---

这篇文章翻译自[Dave Hyatt](http://en.wikipedia.org/wiki/Dave_Hyatt)发布在webkit博客中一系列介绍webcore渲染原理的文章，原文链接如下，

[https://www.webkit.org/blog/115/webcore-rendering-ii-blocks-and-inlines/](https://www.webkit.org/blog/115/webcore-rendering-ii-blocks-and-inlines/)

在前面一篇文章中我描述了CSS框模型的基本结构。在这篇文章中我将描述RenderBox的子类，来帮助读者了解WebCore如何实现block和inline两种布局方式。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# 概念

block flow框布局 - 用来包含行元素(如 paragraph)和其他竖直排列的block。常用的元素p和div都是block流元素。

inline flow框布局 - 用来作为行(line)的一部分存在。常见的元素a,b,i,span都是inline流元素。

在WebCore中有三种渲染器类型涵盖了block和inline flow。他们是RenderBlock，RenderInline和他们共同的父类RenderFlow(现在变成了RenderBoxModelObject)。

RenderFlow.h  
[RenderBlock.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderBlock.h)  
[RenderInline.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderInline.h)  
[RenderBoxModelObject](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderBoxModelObject.h)  

inline flow可以通过修改样式表的方式更改为block flow，反之亦然。

{% highlight css %}
div { display: inline }
span { display: block }
{% endhighlight %}

还有另外一种元素既可以作为block又可以作为inline：replaced元素。一个replaced元素是没有样式表控制它的渲染。它的内容由元素自身定义。例如image，form control, iframe, plugin和applets。

replaced元素可以是block级别也可以是inline级别。当一个replaced元素作为一个block，它会像其他block元素一样竖直排列。当replaced元素作为inline时，它则作为段落的一个部分，内嵌在一行之中。

Images, plugins, frame和applets都继承自一个公共子类来实现replaced元素对象。这个类是RenderReplaced。

Form控件是一个特例，它是replaced元素，但是它实际上继承自RenderBlock。是否为replaced元素不是通过实现类型来判断，而取决于RenderObject上面的一个位。可以通过isReplaced方法查询该对象是否为replaced元素。

{% highlight cpp %}
bool isReplaced() const
{% endhighlight %}


[RenderReplaced.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderReplaced.h)
