---
layout: post
title:  "跨浏览器兼容开发最佳实践"
date:   2014-06-03 00:00:00
categories: Compatibility
---

# 简介

90年代浏览器的出现推动了互联网的发展，互联网同时也在促使浏览器不断演化，虽然为w3c组织通过标准定义使浏览器行为尽量保持一致么，但由于开发团队的不同和不同版本对标准支持的差异，使得跨浏览器仍然是互联网开发的一个重要课题。这篇文章中我们主要介绍一些开发跨浏览器兼容的最佳实践。希望能够对从事跨浏览器应用的开发人员对此有整体的把握，节省些对该问题的研究时间。

# 了解跨浏览器兼容的主要问题

1.   显示问题
    +   浏览器 HTML 渲染处理差异
    +   浏览器 CSS 排版处理差异
    +   功能问题
2.   DOM 接口支持差异
    +   JavaScript 语言支持差异
    +   BOM 浏览器自定义功能差异
    +   浏览器底层实现差异：网络层，插件机制，图形渲染，系统接口等
3.   性能问题
    +   渲染引擎
    +   脚本引擎
    +   网络下载

# 不强求跨浏览器显示一致

网站内容第一，形式第二。
首先确定跨浏览器内容的一致性作为基线，然后逐步优化形式使其趋于一致。
由于浏览器差异性确实存在，不必强求显示效果完全一致。

# 使用稳定模板库简化编程

稳定的模板库可以简化跨浏览器支持工作，很多规则都在模板库中实现。
例如[Html5Boilerplate](http://html5boilerplate.com/)/[Bootstrap](http://getbootstrap.com/)就是不错的选择，它们都提供了跨浏览器的兼容性支持。

# 基于稳定的Web标准编程

尽量避免使用浏览器的特有功能，这些功能的使用是跨浏览器实现的一大障碍。
新的Web标准通常会包含很酷的功能，但是新标准和可能会经过一系列更改变为稳定版本，因此考虑跨浏览器网站实现注意对新标准采取保守态度，尽量基于稳定的标准编程。

# 声明doctype

<!DOCTYPE>是html文档开头的html版本声明，没有<!DOCTYPE>声明浏览器默认选择Quirks模式显示网页。
HTML4.01有三种类型的<!DOCTYPE>

+   HTML4.01 Strict
{% highlight html %}
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd"> 
{% endhighlight %}
+   HTML4.01 Transitional
{% highlight html %}
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd"> 
{% endhighlight %}
+   HTML4.01 Frameset
{% highlight html %}
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
{% endhighlight %}

HTML5只有一种类型
{% highlight html %}
<!DOCTYPE html>
{% endhighlight %}

# 更新现有代码以提供跨浏览器支持

IE7 或IE8 中正常运行的网页可以通过以下方法短时间内在IE9 中使用：
为每个网页添加兼容性模式的 meta 元素，以强制IE9 呈现与传统IE页面类似的页面。
{% highlight html %}
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" /> 
{% endhighlight %}

自动添加兼容性，方法是配置 Web 服务器，以向每个页面发送等同于 meta 元素的自定义 HTTP 响应标头。
{% highlight html %}
  <add name="X-UA-Compatible" value="IE=EmulateIE7" />
{% endhighlight %}

# 验证html和CSS

在开发过程中可以采用一系列的工具对页面进行标准的验证，风格的优化。如果使用了Visual Studio集成开发环境还可以将这些工具集成到build简化操作。

+   [HTML validators](http://validator.w3.org/) – HTML验证工具
+   [CSS validators](http://jigsaw.w3.org/css-validator/) – CSS验证工具
+   [Uglify](https://github.com/mishoo/UglifyJS) – JavaScript优化/压缩工具
+   [JSHint](https://github.com/jshint/jshint/) – JavaScript优化工具

# 使用跨浏览器兼容脚本库

如果需要通过JavaScript操作页面上DOM元素，可以考虑采用跨浏览器脚本库来实现操作，这些脚本库经过了大规模的跨浏览器应用测试，比起自己编写兼容性脚本会节省大量开发和测试时间。
+   [JQuery](http://jquery.com/)
+   [Prototype](http://prototypejs.org/)
+   [MooTools](http://mootools.net/)

# 使用功能检测

+   功能检测：在使用功能之前测试浏览器是否支持该功能。 功能检测使跨浏览器的代码能够发挥作用，无需提前了解每个浏览器的功能。 例如，jQuery 框架基本上完全依赖于功能检测。常用功能检测脚本库modernizr。 

相对于功能检测，我们应该避免使用以下方式，
+   检测特定浏览器：使用浏览器的标识（例如 navigator.userAgent）来更改页面的行为。 
+   假定无关的功能：对一个功能执行功能检测后又使用不同的功能。

[http://jsfiddle.net/aaronzhcl/VurBK/](http://jsfiddle.net/aaronzhcl/VurBK/)

# 合理处理失败场景

我们无法预知客户通过什么浏览器在访问网站，例如客户浏览器可能不支持flash，不支持JavaScript或者特殊的CSS。网站在这种极端情况下也需要能够正常输出必要的内容。
可以禁用了脚本和样式表测试网页，检查是否主要的内容是否仍然可用。