---
layout: post
title:  "CSS性能优化指南"
date:   2014-06-14 00:00:00
categories: Performance
---

一般研究网页性能问题多数时候结论是网络下载，或者javascript操作性能，大多数时候CSS不会成为性能瓶颈。但是本着一个程序员的内心，还是希望每行代码都有好的性能，这篇文章来总结一些常见的CSS性能相关的优化实践，不为面面俱到，希望能够从浏览器角度解释这些不同的做法如何造成性能的差异。

![css](/assets/images/posts/wordle-css.jpg)

<!--more-->

# 避免页面内嵌样式表

内嵌样式表是指将样式表写在html页面的style tag里面。像下面这样，

{% highlight html %}
<head>
<style>
hr {color: sienna;}
p {margin-left: 20px;}
body {background-image: url("images/background.gif");}
</style>
</head>
{% endhighlight %}


建议不将样式表内嵌在html页面的原因在于，

*   设计与内容不能实现分离
*   页面因此体积变大
*   没有有效利用http缓存
*   页面之间样式不能重用
*   维护成本提高

# 避免行间(inline)样式

行间样式表是指html页面元素中使用style属性来指定元素的样式。就像下面这样，

{% highlight html %}
<p style="color:sienna;margin-left:20px;">This is a paragraph.</p>
{% endhighlight %}

这种方式可以确保元素样式被优先应用于元素，然而同时也使得页面与样式耦合，带来与内嵌样式表一样的后果。一般需要应用inline样式可以使用javascript动态的进行设置。

# 在HEAD元素中引用样式表文件

外部样式表文件要在html页面head元素中引用，一般放在tile元素之前，以确保样式表能够在可见元素之前。原因与浏览器渲染原理相关：浏览器样式表来构建渲染树，然后计算页面布局，再进行页面渲染。

![Webkit Workflow](/assets/images/posts/webkitflow.png)

如果样式表文件被放在页面底部，浏览器无法确定所有元素的最终样式，从而无法进行元素渲染。也有浏览器会根据已经加载的样式先进行渲染，但后续的样式会使得页面样式重新布局或重绘，影响浏览体验。

# 避免使用 #import

引用样式表有两种方式，一种是使用link tag，另外一种是使用@import。两种方法的区别在于浏览器会马上下载link tag中的样式表，而import方法会将样式表的下载延迟到文档渲染之后。当然@import也有它存在的意义，例如@import支持[media queries](http://drafts.csswg.org/mediaqueries3/#media0)。但从性能角度考虑，建议使用link tag引用样式表文件。

Steve Souders[这篇文章](http://www.stevesouders.com/blog/2009/04/09/dont-use-import/)详细分析了import在不同条件下对性能的影响。

# 避免使用复杂的选择器

CSS有[多种选择器](http://www.w3schools.com/cssref/css_selectors.asp)以使样式表可以简单明确的确定样式所要应用的元素。

这里有个[CSS选择器的测试页面](http://www.w3schools.com/cssref/trysel.asp)，可以帮助理解不同的选择器规则。

对于CSS选择器一个常见的问题是过量使用ID选择器。由于ID选择器只能应用在对应ID的元素上，不能够被重用。例如下面的选择器，只能应用在#wrapping-element下面包含child-element class元素上。

{% highlight css %}
#wrapping-element .child-element
{% endhighlight %}

一个更好的方式可以使用一个更具有描述性的class名。

{% highlight css %}
.wrapping-child-element
{% endhighlight %}

另一种常见的选择器性能问题在ID选择器前面使用其他选择器，因为ID选择器已经确定了唯一一个元素，在其前面添加其他选择器没有任何用处，仅仅让浏览器多做了一些解析工作。

{% highlight css %}

// CSS规则，从坏到好

div#id1 #id2 div,

#id1 #id2 div,

#id2 div,

.style1 div
{% endhighlight %}

避免使用过长的选择器，浏览器从右到左处理选择器，首先选择所有符合最后选择器的元素，然后再在其中选择符合其左边选择器的元素，依次类推。过长的选择器增加了浏览器处理负担，还不使用一个更具描述性的class名代替。

{% highlight css %}
.class1 .class2 ul.class3 > li.class4,
.class1 .class2 ul.class3 > li.class5,
.class1 .class2 ul.class3 > li.class6,
.class1 .class2 ul.class3 > li.class7
{
     /* Properties */
}

.class1213
{
     /* Properties */
}
{% endhighlight %}

# 使用CSS Reset

每个浏览器都定义了自己的默认样式，比如在主流桌面浏览器中的默认字体各不相同。如果希望页面在不同的浏览器中具有相同的显示效果，常用的方法是使用CSS Reset库来将各个浏览器的CSS重置。

但这个性能有什么相关？因为通常情况下使用CSS Reset库可以简化CSS规则，减小CSS文件大小。

常用的CSS Reset库如下，

*   [Eric Meyer’s “Reset CSS” 2](http://meyerweb.com/eric/tools/css/reset/)
*   [HTML5 Doctor CSS](http://html5doctor.com/html-5-reset-stylesheet/)
*   [Yahoo! (YUI 3) Reset](http://yuilibrary.com/yui/docs/cssreset/)
*   [Normalize.css](https://github.com/necolas/normalize.css)


