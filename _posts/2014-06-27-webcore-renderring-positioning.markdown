---
layout: post
title:  "WebKit渲染 - 定位"
date:   2014-06-27 00:00:00
categories: Mechanism
---

这篇文章翻译自[Dave Hyatt](http://en.wikipedia.org/wiki/Dave_Hyatt)发布在webkit博客中一系列介绍webcore渲染原理的文章，原文链接如下，

[https://www.webkit.org/blog/117/webcore-rendering-iv-absolutefixed-and-relative-positioning/](https://www.webkit.org/blog/117/webcore-rendering-iv-absolutefixed-and-relative-positioning/)

样式表中通过position定位对象相对容器的相对位置。有四种值可以选择，static，absolute，fixed，relative。static是默认值，代表对象使用默认的block定位方式，即没有定位，元素出现在正常的流中（忽略 top, bottom, left, right 或者 z-index 属性）。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# 相对定位(relative)

相对定位可以设置left, top, right, bottom属性，除此之外与static定位完全相同。可以用isRelPositioned方法来判断对象是否使用相对定位。

{% highlight cpp %}
bool isRelPositioned() const
{% endhighlight %}

另外通过以下方法可以查询对象的x,y偏移量。

{% highlight cpp %}
int relativePositionOffsetX() const;
int relativePositionOffsetY() const;
{% endhighlight %}

相对定位只是比静态定位多了一步绘图时的转换。对于布局来说，相对定位对象的偏移是相对其本来应该在的位置而言的。例如下面实例，span采用相对定位，可以看到定位元素的偏移是根据对象原有位置产生的。

{% highlight html %}
<div style="border:5px solid black; padding:20px; width:300px">
Here is a line of text.
<span style="position:relative;top:-10px; background-color: #eeeeee">
This part is shifted<br> up a bit,
</span>
but the rest of the line is in its original position.
</div>
{% endhighlight %}

<div style="border:5px solid black; padding:20px; width:300px; margin-left:auto; margin-right:auto">
Here is a line of text.  <span style="position:relative;top:-10px; background-color:#eeeeee">This part is shifted<br> up a bit</span>, but the rest of the line is in its original position.
</div>

# 绝对和固定定位 (absolute and fixed)

固定(fixed)位置对象的位置是相对界面(viewport，如浏览器窗口)而言的。绝对定位对象的位置相对容器而言，即渲染树中最近的非static定位的父节点。如果没有这样的容器存在，则使用最外层初始的容器RenderView。上一篇文章对容器有详细的介绍。

isPositioned方法可以用来判断渲染器是否是绝对定位(absolute)还是固定定位(fixed)。

{% highlight cpp %}
bool isPositioned() const
{% endhighlight %}

当一个对象是绝对或者固定定位时，它会变成block流，即使是display属性设置为inline/inline-block/table。这时isInline方法返回false。

下列方法可以得到原始和当前display的属性值。布局时有些情况下需要查询原始的display属性。 

{% highlight cpp %}
EDisplay display() const;
EDisplay originalDisplay() const;
{% endhighlight %}

# 定位对象列表

每个容器块都维护一个绝对和固定定位对象的列表。定位这些对象是容器块的职责之一。下面方法可以用来管理定位对象的列表。

{% highlight cpp %}
void insertPositionedObject(RenderObject*);
void removePositionedObject(RenderObject*);
{% endhighlight %}

如果只想布局容器块中的定位对象，可以调用layoutOnlyPositionedObjects方法。如果只有定位对象发生了更改，该方法会返回true。这样就可以忽略非定位对象的布局从而快速返回提升性能。

{% highlight cpp %}
bool layoutOnlyPositionedObjects
{% endhighlight %}

layoutPositionedObjects方法负责定位对象的位置布局。它接收一个布尔变量作为参数，代表是否需要对所有对象重新布局。重新布局在很多情况下需要被用到，后续文章我们会详细描述。

{% highlight cpp %}
bool layoutPositionedObjects(bool relayoutChildren)
{% endhighlight %}

# 定位对象的坐标

定位对象的坐标是相对容器块的padding边缘来说的。例如指定一个left和top均为0的绝对定位对象，其结果是该对象被放在容器块的紧贴左上角边缘之内。下面是个实例。

{% highlight html %}
<div style="position:relative;border:5px solid black;width:300px;height:300px;">
<div style="position:absolute;width:200px;height:200px;background-color:purple"></div>
</div>
{% endhighlight %}

<div style="position:relative;border:5px solid black;width:300px;height:300px;margin-left:auto;margin-right:auto">
<div style="position:absolute;width:200px;height:200px;background-color:purple"></div>
</div>

在WebCore中，坐标偏移总是相对边框(border)边缘而言的，所以上面实例中对象的位置实际上是(5,5)。
