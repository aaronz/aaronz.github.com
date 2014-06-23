---
layout: post
title:  "WebKit渲染 - 基础"
date:   2014-06-23 00:00:00
categories: Mechanism
---

这是描述WebCore渲染系统系列文章的第一篇，我将他发在博客里，同时可以通过WebKit网站的文档访问。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# DOM树

一个网页被解析成的节点树我们称之为文档对象模型(Document Object Model)，简称DOM。这些节点在WebCore中的基类为Node.

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




# 渲染树的销毁

# 访问样式信息

# CSS框对象模型





