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

# inline block

CSS中最令人疑惑的属性要数inline-block. Inline blocks是指可以放置在行内的block流。从外部来看他们是inline replaced元素，从内部来看他们就是正常的block流。CSS中可以通过display属性设置元素为inline block。Inline blocks的isReplaced方法返回true。

{% highlight css %}
div { display: inline-block }
{% endhighlight %}

# Table

Table元素默认为block流。然而我们可以通过CSS属性将其变为inline。

{% highlight css %}
table { display: inline-table }
{% endhighlight %}

从外部来看inline-table是一个inline replaced元素(isReplaced() = true)，但其内部仍然是一个正常的table。

在WebCore中通过RenderTable类表示table。它继承自RenderBlock，具体原因我们在后续讲解定位的章节详述。

[RenderTable.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderTable.h)

# Text

行间原始文本通过RenderText代表，Text在WebCore中永远为inline，因为它只能被至于行内。

[RenderText.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderText.h)

# 获得block和inline信息

获得block或inline状态的一个基本方法是isInline。这个方法可以得到当前对象是否**被设计为行内**对象。这个方法会忽略元素的内部实现是什么(例如text, image, inline, inline-block, inline-table)。

{% highlight cpp %}
bool isInline() const
{% endhighlight %}

大多数人经常烦的一个的错误是假设isInline代表一个对象是否永远是inline流，text或者是replaced元素。实际上inline-block和inline-table元素也会放回true。

要看一个对象实际上是否为block或者inline，应该使用下列方法。

{% highlight cpp %}
bool isInlineFlow() const
bool isBlockFlow() const
{% endhighlight %}

这些方法依赖于对象的内部实现。例如一个inline-block实际上仍然是一个block流而不是inline流。设置为inline-block的元素外部表现是inline，内部仍然是block流。

针对类型的block和inline查询应该通过下面方法。

{% highlight cpp %}
bool isRenderBlock() const
bool isRenderInline() const
{% endhighlight %}

isRenderBlock方法在定位时非常有用，因为block流和table都是可定位容器。

如果需要知道一个对象是否是inline block或者inline table，可以使用下面方法。

{% highlight cpp %}
bool isInlineBlockOrInlineTable() const
{% endhighlight %}

# block流的子代

block流的子代都遵循一条不变的规则如下，

所有block流子代元素必须是block，或者所有的block流子代元素必须是inline。

换句话说，除了浮动(float)元素和定位(position)元素，所有的子代元素isInline方法必须全返回true或者全返回false。

要查询子代元素是否为inline或者block，可以通过childrenInline方法。

{% highlight cpp %}
bool childrenInline() const
{% endhighlight %}

# inline流的子代

inline流的子代规则相对block流的规则更为简单，即

所有inline流的自带元素必须是inline。

# 匿名block

为了让所有block流的子代元素都遵守子代规则，渲染树会创建匿名block。

{% highlight html %}
<div>
    Some text
    <div>
        Some more text
    </div>
</div>
{% endhighlight %}

上面的实例代码中外部的div包含两个子元素，Some text和另外一个div。第一个子元素是一个inline元素，但是第二个元素是一个block元素。因为要遵守block流的规则，渲染树会创建一个匿名block流来包含Some text。因此实际上的渲染树就会变成下面这样。

{% highlight html %}
<div>
    <anonymous block>
        Some text
    </anonymous block>
    <div>
        Some more text
    </div>
</div>
{% endhighlight %}

要判断一个渲染器是否为一个匿名block可以通过isAnonymousBlock来查询。

{% highlight cpp %}
bool isAnonymousBlock() const
{% endhighlight %}

当一个block流中包含inline的子元素，然后一个block元素插入到了子元素列中，这时候匿名block就会被创建来包含这些inline元素。连续的inline元素会被包含在同一个匿名block中。执行这个动作的方法是RenderBlock中的makeChildrenNonInline。

{% highlight cpp %}
void makeChildrenNonInline(RenderObject *insertionPoint)
{% endhighlight %}

# inline流内部的block

一个让人郁闷的实现是block元素被放置在inline流中。例如下面示例，

{% highlight html %}
<i>Italic only <b>italic and bold
<div>
    Wow, a block!
</div>
<div>
    Wow, another block!
</div>
More italic and bold text</b> More italic text</i>
{% endhighlight %}

两个div违背的b元素inline规则。渲染树要做一系列复杂的事情来完成对树的修复使其符合规则。三个匿名的block会被创建，第一个block用来包含div之前所有的inline内容。第二个匿名block会用来存放div。第三个匿名block用来包含div之后的所有inline内容。最终的结果如下，

{% highlight html %}
<anonymous pre block>
<i>Italic only <b>italic and bold</b></i>
</anonymous pre block>
<anonymous middle block>
<div>
Wow, a block!
</div>
<div>
Wow, another block!
</div>
</anonymous middle block>
<anonymous post block>
<i><b>More italic and bold text</b> More italic text</i>
</anonymous post block>
{% endhighlight %}

注意i元素和b元素被拆分为了两个渲染对象，他们都被包含在第一个和第三个匿名block中。渲染树最终通过continuation chain来将他们串联在一起。

{% highlight cpp %}
RenderFlow* continuation() const
bool isInlineContinuation() const
{% endhighlight %}

b的渲染器可以通过b元素的renderer()方法得到，然后通过该渲染器的continuation()方法可以得到后续匿名block的渲染器，再通过该后续渲染器的continuation()方法得到最后一个匿名block的渲染器。

RenderInline.cpp中的splitFlow方法用来递归的拆分inline流，并为之创建continuation chain连接。
