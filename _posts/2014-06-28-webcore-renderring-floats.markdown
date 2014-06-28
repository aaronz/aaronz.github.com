---
layout: post
title:  "WebKit渲染 - 浮动"
date:   2014-06-28 00:00:00
categories: Mechanism
---

这篇文章翻译自[Dave Hyatt](http://en.wikipedia.org/wiki/Dave_Hyatt)发布在webkit博客中一系列介绍webcore渲染原理的文章，原文链接如下，

[https://www.webkit.org/blog/118/webcore-rendering-v-floats/](https://www.webkit.org/blog/118/webcore-rendering-v-floats/)

这篇文章主要介绍WebCore中浮动(float)的涉及原理。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

<div style="float:right; width:50px; height:50px; background-color:purple; margin-left: 5px"></div>
浮动可以控制对象向段落左边或者向段落右边浮动。然后段落的各行就会围绕着浮动对象排布。当前这个段落里面就是一个浮动框对象。这个紫色的浮动框浮动于右上角。所有的文字围绕它排布。

{% highlight html %}
<div style="float:right; width:50px; height:50px; background-color:purple; margin-left: 5px"></div>
{% endhighlight %}

HTML本身也有语法中隐含了浮动的概念。例如，img元素的align属性可以用来控制一个图片的浮动。

{% highlight html %}
<img align=left src="...">
{% endhighlight %}

<div style="float:left;width:50px;height:100px; background-color:orange; margin-right:5px"></div>
浮动可以跨多个段落。当前这个段落的示例中可以看到浮动虽然声明在段落之中，它却足够高到超出该段落而扩张到了下个段落。

因为浮动可以影响到多个块，WebCore在block流中采用浮动对象列表来追踪所有介入到该块中的浮动对象。因此一个浮动对象可能出现在多个block的浮动对象列表中。line布局的对象需要了解浮动对象的位置然后使内容可以避开浮动对象。浮动对象列表就是可以让block更容易地访问框内部的浮动对象的位置，以便知道如何绕过这些浮动对象。

浮动对象列表包含下列数据结构，

{% highlight cpp %}
struct FloatingObject {
    enum Type {
        FloatLeft,
        FloatRight
    };

    FloatingObject(Type type)
        : node(0)
        , startY(0)
        , endY(0)
        , left(0)
        , width(0)
        , m_type(type)
        , noPaint(false)
    {
    }

    Type type() { return static_cast<type>(m_type); }

    RenderObject* node;
    int startY;
    int endY;
    int left;
    int width;
    unsigned m_type : 1; // Type (left or right aligned)
    bool noPaint : 1;
};

{% endhighlight %}

从上面代码中可以看出数据结构中包含了一个矩形的信息(top, bottom, left, width)。原因是浮动框的位置和大小并不依赖于保存浮动对象列表的框。这样保存浮动对象列表的框对象可以很容易的查询每个浮动对象在自己的坐标系中的位置信息。

另外浮动对象的margin也包含在这个矩形内，因为段落行不仅仅避免浮动框的边框，还要避免浮动框的margin。

下面的方法可以用来操作浮动对象列表，

{% highlight cpp %}
void insertFloatingObject(RenderObject*);
void removeFloatingObject(RenderObject*);
void clearFloats();
void positionNewFloats();
{% endhighlight %}

前两个方法的作用不言而喻，用来从浮动对象列表中添加和删除浮动对象。clearFloats用来删除所有的浮动对象列表中的浮动对象。

当一个对象被添加到列表时，它的位置信息是没有设置的。竖直坐标是-1。positionNewFloats方法在布局过程中被调用来设置所有浮动对象的位置信息。CSS有[很多规则](http://www.w3.org/TR/CSS21/visuren.html#propdef-float)来设置浮动框的走向。正是这个方法保证这些规则都被正确的遵守。

# 清除 (clearance)

<span style="width:50px;height:100px;background-color:blue; margin-right: 5px; float:left"></span>
CSS为对象提供了一种方式来决定他们是否要位于所有的左浮动框，右浮动框还是所有的浮动框的下面(即不环绕浮动框)。clear属性可以通过设置none，left，right，both来控制。

<p style="clear: left">
这个段落就设置了clear: left，从而使得这个段落能够置于浮动框之下。清除可以应用在block和inline的对象。同时也可以被应用在浮动对象上，以保证该浮动对象处于其他浮动对象之下。
</p>