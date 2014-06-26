---
layout: post
title:  "WebKit渲染 - 布局简介"
date:   2014-06-24 00:00:00
categories: Mechanism
---

这篇文章翻译自[Dave Hyatt](http://en.wikipedia.org/wiki/Dave_Hyatt)发布在webkit博客中一系列介绍webcore渲染原理的文章，原文链接如下，

[https://www.webkit.org/blog/116/webcore-rendering-iii-layout-basics/](https://www.webkit.org/blog/116/webcore-rendering-iii-layout-basics/)

当渲染器刚开始被创建加入到渲染树中时，它们还没有坐标和大小的信息。为所有的渲染器计算坐标和大小等几何属性的过程我们称之为布局(layout)。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# 概念

所有的渲染器都包含一个layout方法。

{% highlight cpp %}
void layout()
{% endhighlight %}

布局是一个递归的过程。一个叫做FrameView的类代表了一个文档的的视图，它也包含一个layout方法。这个frame view负责管理整个文档渲染树的layout。

FrameView有两种layout方式。第一种也是最常见的一种是整体布局。即从树的根节点的layout方法开始调用，从而整棵树布局都被更新。第二种方式是使布局发生在子树范围之内。它用作仅需要特定子树重新布局而不会影响周围节点的情况。现在子树布局仅在一些文本域使用。

# Dirty 位

布局使用dirty位机制来决定一个对象是否需要重新布局。每当新的渲染器加入到渲染树后，它们将自己和相关的父节点的dirty位设置为true。渲染树通过三个位变量来判断是否需要布局。

{% highlight cpp %}
bool needsLayout() const { return m_needsLayout || m_normalChildNeedsLayout ||
                                  m_posChildNeedsLayout; }
bool selfNeedsLayout() const { return m_needsLayout; }
bool posChildNeedsLayout() const { return m_posChildNeedsLayout; }
bool normalChildNeedsLayout() const { return m_normalChildNeedsLayout; }
{% endhighlight %}

第一个位变量用来判断渲染器自身是否dirty，可以通过selfNeedsLayout来查询该值。当这个bit设置为true，相关的前驱渲染器会设置一个位来表示它存在一个dirty的子节点。这个为变量依赖于前驱结点的定位状态。posChildNeedsLayout代表定位子节点是否dirty。normalChildNeedsLayout代表一个处于流中的子节点是否dirty。区分定位节点和流中的节点，可以优化定位节点的布局过程。(由于定位节点位置固定，所以不会影响前驱和后继节点的几何属性)

# 容器块(containing block)

前面提到的前驱结点到底是指什么？当一个对象被标识为需要布局时，dirty位相应被置为true的节点链被称为容器块(containing block)。容器框被用作子节点的布局的坐标空间。渲染器有xPos和yPos坐标，这些坐标是以其容器块为基准的。但究竟如何定义一个容器块呢？

[这里有CSS 2.1 标准说明书中对这个概念的介绍](http://www.w3.org/TR/CSS21/visuren.html#containing-block)。

我从渲染树角度对这个概念的解释如下，

渲染器的容器框是指决定子代渲染器位置信息的父节点框。换句话说当布局发生时，容器块会负责定位所有的子节点渲染器。

渲染树的根节点叫做RenderView，根据CSS2.1这个类被作为初始的容器框。它也是在Document级别调用renderer()返回的渲染器。

[RenderView.h](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderView.h)

RenderView这个初始容器框永远根据当前可见窗口来设置大小。在桌面浏览器上，其大小相当于浏览器窗口可见区域的大小。这个容器框永远被置为文档的(0,0)坐标点上。下图描绘了文档的初始容器块所在的区域。黑框内代表RenderView，灰色的部分代表整个文档。

<center><p></p>
<div style="width:100px;height:300px; background-color:#dddddd">
<div style="border:3px solid black;width:94px;height:94px">
</div>
</div>
<p></p></center>

当文档滚动时，初始容器块会移出界面。他永远在文档的上沿并保持界面的大小。这里容易让人产生疑惑之处在于人们经常想象容器块虽然保持在界面中但是已经超出了文档区域。

[这里是CSS 2.1标准说明书中对容器块的详细介绍。](http://www.w3.org/TR/CSS21/visudet.html#containing-block-details)

其规则可以总结如下，

+   根节点的渲染器永远用RenderView作为其容器块。
+   如果渲染器设置了相对(relative)或静态(static)位置信息，那与之相对应的容器块为渲染树中最近的block级别的前驱结点。
+   如果渲染器设置了固定(fixed)位置信息，那么容器块就是RenderView。RenderView负责调整固定位置元素的坐标，解释文档滚动时的位置信息。


