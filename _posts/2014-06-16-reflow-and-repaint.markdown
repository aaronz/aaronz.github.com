---
layout: post
title:  "浏览器布局与重绘 - reflow & repaint"
date:   2014-06-16 00:00:00
categories: Mechanism
---

浏览器渲染过程中有两个重要的概念，他们与页面渲染性能休戚相关，这两个概念是布局(reflow)与重绘(repaint)。本文将结合WebKit的具体实现来解释这两个概念，并介绍与之相关的编程指导经验。

![paint](/assets/images/posts/paint.jpg)

<!--more-->

# 概念

+   布局(reflow) - 浏览器构建渲染树完成时不包含位置和大小信息。计算元素位置和其他几何信息的过程称为布局。

	*   Reflow采用基于流的布局模型，大多数情况下一次遍历就能计算出几何信息。处于流中靠后位置元素通常不会影响靠前位置元素的几何特征，因此布局可以按从左至右、从上至下的顺序遍历文档。但是也有例外情况，比如 HTML 表格的计算就需要不止一次的遍历。
	*   Reflow是一个递归的过程。从根呈现器(对应于 HTML 文档的 <html> 元素)开始，递归遍历部分或所有的框架层次结构，为每一个需要计算的呈现器计算几何信息。

+   重绘(repaint) - 当布局结束后，浏览器遍历呈现树，调用呈现器的paint方法，将呈现器的内容显示在屏幕上。
	
	部分呈现器发生了更改，但是不会影响整个树。更改后的呈现器将其在屏幕上对应的矩形区域设为无效，这导致 OS 将其视为一块“dirty 区域”，并生成“paint”事件。OS 会很巧妙地将多个区域合并成一个。

以下是一个wikipedia网站布局过程录像，
<p><object width="480" height="400" align="middle" data="http://player.youku.com/player.php/sid/XMzI5MDg0OTA0/v.swf" type="application/x-shockwave-flash"><param name="src" value="http://player.youku.com/player.php/sid/XMzI5MDg0OTA0/v.swf" /><param name="allowfullscreen" value="true" /><param name="quality" value="high" /><param name="allowscriptaccess" value="always" /></object></p>

# 浏览器实现

## 布局

我们可以参考WebKit的实现来解释布局的逻辑。在WebKit中页面最终会被解析成为渲染树，渲染树的节点通过[RenderObject](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderObject.h)定义。其中包含了以下常用遍历渲染树的方法，

{% highlight cpp %}
RenderObject* firstChild() const;
RenderObject* lastChild() const;
RenderObject* previousSibling() const;
RenderObject* nextSibling() const;
{% endhighlight %}

对于所有元素，页面会为其构建相应的RenderBox,也就是常说的[CSS框对象模型](http://www.w3.org/TR/CSS21/box.html#box-dimensions)。

![CSS框对象模型](/assets/images/posts/boxdim.png)

每个对应的[RenderBox](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderBox.h)都有一个layout方法。用以在渲染树中递归调用，以完成整体布局。下面代码是RenderBox的继承类[RenderBlock](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderBlock.cpp)实现的layout方法。

{% highlight cpp %}
void RenderBlock::layout()
{
    StackStats::LayoutCheckPoint layoutCheckPoint;
    OverflowEventDispatcher dispatcher(this);

    // Update our first letter info now.
    updateFirstLetter();

    // Table cells call layoutBlock directly, so don't add any logic here.  Put code into
    // layoutBlock().
    layoutBlock(false);
    
    // It's safe to check for control clip here, since controls can never be table cells.
    // If we have a lightweight clip, there can never be any overflow from children.
    if (hasControlClip() && m_overflow && !gDelayUpdateScrollInfo)
        clearLayoutOverflow();

    invalidateBackgroundObscurationStatus();
}
{% endhighlight %}

下面是RenderBlock的layout方法在布局过程中被递归调用的调用栈。这个时刻正在计算并设置元素的margin。

{% highlight text %}
ChildEBP RetAddr  
0018d868 020ffd0a WebKit!WebCore::LayoutBoxExtent::setBefore
0018d880 01fc53b9 WebKit!WebCore::RenderBox::setMarginBefore+0x3a
0018d8b4 01ff3a01 WebKit!WebCore::RenderBox::updateLogicalHeight+0x69
0018d8d0 01ff27fc WebKit!WebCore::RenderBlockFlow::updateLogicalHeight+0x11
0018da28 01fd63b9 WebKit!WebCore::RenderBlockFlow::layoutBlock+0x3bc
0018da48 01ff43a4 WebKit!WebCore::RenderBlock::layout+0x49
0018daf8 01ff391f WebKit!WebCore::RenderBlockFlow::layoutBlockChild+0x244
0018db40 01ff2693 WebKit!WebCore::RenderBlockFlow::layoutBlockChildren+0x16f
0018dca0 01fd63b9 WebKit!WebCore::RenderBlockFlow::layoutBlock+0x253
...
0018eba8 01ff43a4 WebKit!WebCore::RenderBlock::layout+0x49
0018ec58 01ff391f WebKit!WebCore::RenderBlockFlow::layoutBlockChild+0x244
0018eca0 01ff2693 WebKit!WebCore::RenderBlockFlow::layoutBlockChildren+0x16f
0018ee00 01fd63b9 WebKit!WebCore::RenderBlockFlow::layoutBlock+0x253
0018ee20 02044c51 WebKit!WebCore::RenderBlock::layout+0x49
0018ee2c 02041963 WebKit!WebCore::RenderView::layoutContent+0x41
0018ee70 0224d152 WebKit!WebCore::RenderView::layout+0x323
0018ef6c 02250cb5 WebKit!WebCore::FrameView::layout+0x932
0018f004 01de4662 WebKit!WebCore::FrameView::updateLayoutAndStyleIfNeededRecursive+0x45
0018f088 01dde94f WebKit!WebView::updateBackingStore+0x152
0018f158 01de4e0e WebKit!WebView::paint+0x29f
0018f228 75367834 WebKit!WebView::WebViewWndProc+0x1fe
{% endhighlight %}

## 绘制

绘制的逻辑相对简单，[RenderBlock](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderBlock.cpp)中提供了paint方法。当图像有所更改的时候系统消息会通知页面重绘，从而调用相应元素的paint方法。

{% highlight cpp %}
void RenderBlock::paint(PaintInfo& paintInfo, const LayoutPoint& paintOffset)
{% endhighlight %}

paint方法会接下来调用paintObject方法，其中的注释可以帮助理解绘制的各个步骤。具体逻辑参考[RenderBlock代码](http://trac.webkit.org/browser/trunk/Source/WebCore/rendering/RenderBlock.cpp)。

{% highlight cpp %}
void RenderBlock::paintObject(PaintInfo& paintInfo, const LayoutPoint& paintOffset)
{
    PaintPhase paintPhase = paintInfo.phase;

    // 1. paint background, borders etc
    ...
    // 2. paint contents
    ...
    // 3. paint selection
    ...
    // 4. paint floats.
    ...
    // 5. paint outline.
    ...
    // 6. paint continuation outlines.
    ...
    // 7. paint caret.
    ...
}
{% endhighlight %}

下面是一个paint方法被调用的调用栈。

{% highlight text %}
ChildEBP RetAddr  
0018e980 02042435 WebKit!WebCore::RenderBlock::paintObject
0018e9c4 0202e06d WebKit!WebCore::RenderView::paint+0x115
0018ea5c 0202cece WebKit!WebCore::RenderLayer::paintBackgroundForFragments+0x15d
0018ebb4 0202c6d4 WebKit!WebCore::RenderLayer::paintLayerContents+0x60e
0018ebd0 0202c4fc WebKit!WebCore::RenderLayer::paintLayerContentsAndReflection+0xd4
0018eccc 02026110 WebKit!WebCore::RenderLayer::paintLayer+0x50c
0018ed84 02250591 WebKit!WebCore::RenderLayer::paint+0x70
0018ee34 01eef59e WebKit!WebCore::FrameView::paintContents+0x381
0018ef10 01de4498 WebKit!WebCore::ScrollView::paint+0x1ce
0018eff4 01de472d WebKit!WebView::paintIntoBackingStore+0x148
0018f088 01dde94f WebKit!WebView::updateBackingStore+0x21d
0018f158 01de4e0e WebKit!WebView::paint+0x29f
0018f228 75367834 WebKit!WebView::WebViewWndProc+0x1fe
{% endhighlight %}



https://delicious.com/stubbornella/reflow
