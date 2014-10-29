---
layout: post
title:  "WebKit渲染 - 基础"
date:   2014-06-23 00:00:00
categories: Mechanism
---

这篇文章翻译自[Dave Hyatt](http://en.wikipedia.org/wiki/Dave_Hyatt)发布在webkit博客中一系列介绍webcore渲染原理的文章，原文链接如下，

[https://www.webkit.org/blog/114/webcore-rendering-i-the-basics/](https://www.webkit.org/blog/114/webcore-rendering-i-the-basics/)

这是描述WebCore渲染系统系列文章的第一篇，将他发在博客里，同时可以通过WebKit网站的文档访问。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# DOM树

一个网页被解析成的节点树称之为文档对象模型(Document Object Model)，简称DOM。这些节点在WebCore中的基类为Node.

[Node.h](http://trac.webkit.org/browser/trunk/Source/WebCore/dom/Node.h)

Node可以划分为几种类别如下，

+   Document - 树的根节点永远是document. 有下面三种document类存在，  

	Document - 用于除了SVG documents的所有XML document  
	HTMLDocument - 用于HTML document，继承自Document  
	SVGDocument - 用于SVG document，继承自Document  

	[Document.h](http://trac.webkit.org/browser/trunk/Source/WebCore/dom/Document.h)  
	[HTMLDocument.h](http://trac.webkit.org/browser/trunk/Source/WebCore/html/HTMLDocument.h)  
	[SVGDocument.h](http://trac.webkit.org/browser/trunk/Source/WebCore/svg/SVGDocument.h)  

+   Element - HTML或者XML中所有的Tag元素(带有<>的元素)会被解析成Element。从渲染的角度一个元素是可以被转变为具体子类供渲染引擎查询的节点。

	[Element.h](http://trac.webkit.org/browser/trunk/Source/WebCore/dom/Element.h)

+   Text - Element元素之间的文字会被转化成Text节点。Text节点存储这些文字，渲染树可以查询这些文字节点。
	
	[Text.h](http://trac.webkit.org/browser/trunk/Source/WebCore/dom/Text.h)

# 渲染树

渲染的核心是渲染树。渲染树与DOM树都是一棵对象树，每个对象对应了文档中的一个节点(document/element/text)，但渲染树中可能包含不在dom树中的节点。

渲染树节点的基类为RenderObject.

[RenderObject.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderObject.h)

DOM节点中可以通过Node中render()方法RenderObject。

{% highlight cpp %}
RenderObject* renderer() const
{% endhighlight %}

以下是遍历渲染树的常用方法，

{% highlight cpp %}
RenderObject* firstChild() const;
RenderObject* lastChild() const;
RenderObject* previousSibling() const;
RenderObject* nextSibling() const;
{% endhighlight %}

下面是遍历渲染对象的直接子节点的代码示例，这也是最常用的遍历渲染树的方法，

{% highlight cpp %}
for (RenderObject* child = firstChild(); child; child = child->nextSibling()) {
    ...
}
{% endhighlight %}

# 渲染树的创建

渲染树是通过一个叫的附加(attachment)过程创建的。当一个文档被解析成DOM节点时，会调用一个attach方法将DOM节点附加到渲染树。

{% highlight cpp %}
void attach()
{% endhighlight %}

attach方法会计算DOM节点的样式信息。如果该元素的样式display属性为none，或者该元素是其他display属性为none的子元素，那就不会为该节点创建渲染器。节点的渲染器的类型由节点的类型和节点的display样式来共同决定。

attach是一个自上而下的递归调用过程。父节点的渲染器的创建总是在子节点渲染器创建之前。

# 渲染树的销毁

当节点从文档中移除或者文档关闭(tab或窗口关闭)的时候，节点的渲染器会被销毁。这个时候DOM节点上的detach方法会被调用来断开和销毁渲染器。

{% highlight cpp %}
void detach()
{% endhighlight %}

detach的过程是一个自底向上的递归调用过程。子代的渲染器销毁永远优先于父节点的渲染器销毁。

# 访问样式信息

在attach过程中DOM查询CSS得到元素的样式信息。将其结果存储在RenderStyle对象中。

[RenderStyle.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/style/RenderStyle.h)

通过这个对象可以查询所有WebKit支持的CSS属性。RenderStyle是使用引用计数的对象。如果一个DOM节点创建了渲染器，他会通过setStyle方法将样式信息设置到渲染器。

{% highlight cpp %}
void setStyle(RenderStyle*)
{% endhighlight %}

渲染器添加一个对该样式对象的一个引用，并且会维护至该对象被销毁或者有新的样式替换改样式。

RenderStyle对象可以通过RenderObject的style方法访问。

{% highlight cpp %}
RenderStyle* style() const
{% endhighlight %}

# CSS框对象模型

RenderObject一个重要子类叫做RenderBox。这个类型用来表示符合CSS框对象模型的对象，包括那些可以设置border, padding, margins, width, height的对象。现在有些对象不符合CSS框对象模型但仍然继承自RenderBox，例如SVG对象。但这是一个实现上的错误，将来需要被修正。

下面是来自CSS2.1标准说明书中用来描述CSS框对象模型的图片。

![css box model](/assets/images/posts/boxdim.png)

下面方法可以用来得到页面的border/margin/padding的值。这里不应该使用RenderStyle来查看这些信息，除非希望查看的是对象上原始样式信息，因为真正的RenderObject信息可能与原始信息大相径庭。(特别是针对table对象，因为其可以改写cell padding，并且cell之间的border会被收起。)

{% highlight cpp %}
int marginTop() const;
int marginBottom() const;
int marginLeft() const;
int marginRight() const;

int paddingTop() const;
int paddingBottom() const;
int paddingLeft() const;
int paddingRight() const;

int borderTop() const;
int borderBottom() const;
int borderLeft() const;
int borderRight() const;
{% endhighlight %}

width()和height()方法可以给出框对象的width和height(其中包含了border的宽度)。

{% highlight cpp %}
int width() const;
int height() const;
{% endhighlight %}

client框派出了border和scrollbar的宽度，包含了padding的宽度。

{% highlight cpp %}
int clientLeft() const { return borderLeft(); }
int clientTop() const { return borderTop(); }
int clientWidth() const;
int clientHeight() const;
{% endhighlight %}

content框用来描述派出了border和padding的区域。

{% highlight cpp %}
IntRect contentBox() const;
int contentWidth() const { return clientWidth() - paddingLeft() - paddingRight(); }
int contentHeight() const { return clientHeight() - paddingTop() - paddingBottom(); }
{% endhighlight %}

当框中包含横向或纵向滚动条，滚动条会置于border和padding之间。滚动条的大小被包含在client height和client width之中，不包含在content框中。滚动区域的坐标和大小可以从RenderObject中得到。这部分会在后续介绍滚动的章节详细介绍。

{% highlight cpp %}
int scrollLeft() const;
int scrollTop() const;
int scrollWidth() const;
int scrollHeight() const;
{% endhighlight %}

框也包含x，y的坐标信息。这些位置信息是相对与其前驱结点而言的。这天规则有很多的例外，这也是最容易产生误解的地方。

{% highlight cpp %}
int xPos() const;
int yPos() const;
{% endhighlight %}
