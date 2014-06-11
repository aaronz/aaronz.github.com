---
layout: post
title:  "IE HTML DOM 兼容性变更"
date:   2014-05-08 22:49:54
categories: Compatibility
---

随着Internet Explorer版本的更新，对HTML和DOM的功能有所更改，以便更好的支持现行标准及与其他浏览器行为保持一致。

<!--more-->

IE针对HTML和DOM的主要功能更改如下，

+   [二进制行为和XML架构](#binary-behavior)
+   [内容属性和DOM属性](#content-attribute)
+   [CreateElement方法](#create-element)
+   [IFrame资源管理](#iframe-management)
+   [MIME类型和纯文本内容](#mime-plaintext)
+   [原生XML对象类型](#native-xml)
+   [对象元素回退适用于DOM](#fallback-object)
+   [指针事件更新](#pointer-event)
+   [脚本元素和事件执行](#script-event)
+   [空格保存和DOM](#space-reservation)
+   [Window事件行为更改](#window-event)

<br/>

# 二进制行为和XML架构 <a name="binary-behavior"></a>

IE9开始禁止使用命名空间导入二进制行为，但是可以通过CSS behavior属性注册二进制行为。

使用HTML标记在网页顶部指定行为，但上述代码不能在XML模式中使用。

{% highlight html %}
<html xmlns:myNamespace>
  <?import namespace="myNamespace" implementation = "my.htc">
  ...
  <myNamespace:calendar/>
{% endhighlight %}

可以在XML模式中使用以下代码

{% highlight html %}
<style>
.calendar {
  -ms-behavior: url(my.htc);
}
</style>
...
<div class="calendar"></div>
{% endhighlight %}

# 内容属性和DOM属性 <a name="content-attribute"></a>

IE9内容属性(content-attribute)将不再连接到DOM expando, 这提高了Internet Explorer和其他浏览器之间的互操作性。

内容属性在是HTML源中指定的属性，例如，<element attribute1="value" attribute2="value">。许多内容属性都作为HTML的一部分进行预定义；HTML还支持用户自定义的内容属性。

在以下示例中，id和 class是HTML中预定义的内容属性，myAttr是用户定义的内容属性：
{% highlight html %}
<div id="myElement" class="b" myAttr="custom"></div>
{% endhighlight %}

在以下脚本示例中，id和 className是预定义的属性： 
{% highlight javascript %}
var div = document.getElementById("myElement");
var divId = div.id; // Gets the value of the id content attribute
var divClass = div.className; // Gets the value of the class content attribute
{% endhighlight %}

在IE8及以前版本中，包括IE8标准模式和IE9以前模式，存在myAttr内容属性表示存在myAttr DOM expando，
{% highlight javascript %}
var divExpando = div.myAttr; // divExpando would get the value "custom" in IE8
{% endhighlight %}

IE9中的重大更改是DOM expando不再由用户定义的内容属性的存在来表示，
{% highlight javascript %}
var divExpando = div.myAttr; // divExpando would get an undefined value
{% endhighlight %}

为解决此问题，请使用“getAttribute”API 检索用户定义的内容属性的值。建议所有版本的IE都使用此变通方法，并且新版本与旧版本IE相比不要求特殊外壳。 
{% highlight javascript %}
var divExpando = div.getAttribute("myAttr");
{% endhighlight %}

[http://jsfiddle.net/aaronzhcl/6d57u/](http://jsfiddle.net/aaronzhcl/6d57u/)

# CreateElement方法 <a name="create-element"></a>

自IE9 开始，使用尖括号 (< >) 时，createElement 会触发invalid character error异常。
{% highlight javascript %}
 // Works in IE8-
 document.createElement("<div id='myDiv'>");
 // Works in IE9+
 var elem2 = document.createElement("div");
 elem2.setAttribute("id", "myDiv");
{% endhighlight %}

# Iframe资源管理 <a name="iframe-management"></a>

IE8-会在网页下次处理导航事件期间释放iframe资源。导航事件之前，将继续运行与 iframe 元素（或其内容）关联的代码。

IE9从 DOM 中删除iframe后，将立即释放与 iframe 元素关联的资源。如果在 iframe 或其任何子元素上调用与 DOM 关联的 API，将触发异常并显示消息“不能执行已释放 Script 的代码”。如果需要 iframe 中的对象，先将它们从 iframe 中复制出来，然后从 DOM 中将其删除。 

# MIME类型和纯文本内容 <a name="mime-plaintext"></a>

IE9 标准模式 “text/plain” MIME 类型的文档不会通过 MIME 探查为其他类型。文档仅以纯文本方式呈现或下载。这样可以更轻松地共享 HTML 源代码片段。防止”text/plain”类型的脚本注入攻击。 配置服务器所有文档发送适当的 Content-Type 标头。例如，如果你的服务器提供可移植文档格式 (PDF) 文件以供下载，请确保文件使用 “application/pdf” MIME 类型。

# 原生XML对象类型 <a name="native-xml"></a>

IE9 引入了原生XML对象的概念。原生 XML 对象可以在页面中呈现，并且可以与 HTML 对象支持的相同文档对象模型 (DOM) API 一起使用。 
IE9之前版本通过 Microsoft XML (MSXML) 对象管理 XML，IE9 中仍然包含这些对象。但是本机 XML 对象与 MSXML 对象不兼容。
站点中的代码尝试混合这两种对象时，通常会引发 JavaScript 异常，这可能导致兼容性问题。
例如： xhr.responseXML就是一个MSXML对象

[http://jsfiddle.net/aaronzhcl/9bHnG/](http://jsfiddle.net/aaronzhcl/9bHnG/)

# 对象元素回退适用于DOM <a name="fallback-object"></a>

object 元素含回退内容（通常是 embed 元素）时，IE9 会解析此内容，并且将其包含在文档对象模型 (DOM) 中，之前版本的IE不会执行此操作。

如果 object 元素与其任意一个回退元素具有相同的名称属性，则 window["myName"] 现在会返回所有具有名称 "myName" 的元素的集合。 
在访问关于返回值的方法和属性时，假定IE会返回单个元素（通常是 object 元素）的页面可能导致出现异常。

使用document["myName"]可以正常返回object对象。

[http://jsfiddle.net/aaronzhcl/ad264/](http://jsfiddle.net/aaronzhcl/ad264/)

# 指针事件更新 <a name="pointer-events"></a>

指针事件是一些事件和相关接口，用于处理来自鼠标、手写笔或触摸屏等设备的硬件不可知的指针输入。自从在IE10中首次引入以来，指针事件已成为万维网联合会 (W3C) 规范。
为了符合 W3C 指针事件规范的“候选推荐”，与IE10 相比，IE11 实现已略有更改。

+   MS 供应商前缀删除
+   行为更新

[http://jsfiddle.net/aaronzhcl/8xpQq/](http://jsfiddle.net/aaronzhcl/8xpQq/)

# 脚本元素和事件执行 <a name="script-event"></a>

IE9 标准模式为 script 元素引入了基于标准的 load 事件。以前版本的 IE仅支持 script 元素的不可互操作的 onreadystatechange 事件。
通常这些事件用于页面加载以后执行，以防止阻碍页面加载。在这些情况下，同时为 onreadystatechange 和 load 注册可能导致回调执行两次。这通常导致脚本错误或页面功能损坏。 
建议对这些情形使用基于标准的 load 事件。Web 开发人员应使用功能检测来检测支持 script 元素的 load 事件的浏览器。在传统版IE中，应回滚脚本以使用 onreadystatechange。 

[http://jsfiddle.net/aaronzhcl/6LrdN/](http://jsfiddle.net/aaronzhcl/6LrdN/)

# 空格保存和DOM <a name="space-reservation"></a>

在使用 IE9 时，添加到网页的任何空格仍会在 DOM 中存在。
如果希望看到类似 IE8 的行为，请使用元素遍历 API（例如，firstElementChild）。

[http://jsfiddle.net/aaronzhcl/7xBz6/](http://jsfiddle.net/aaronzhcl/7xBz6/)

# Window事件行为更改 <a name="window-event"></a>

IE11更改了window.event 行为。 对于 IE11 边缘模式：

+   如果没有为事件分配处理程序，那么传递给 window.event 属性的处理程序的对象将返回 "undefined"。 （在之前的文档模式中，在此情况下它会返回 NULL。） 
+   传递给 window.event 的处理程序的对象类型已更改。 对于 IE11 边缘模式，这是一个 Event 对象。 （在之前的文档模式中，这是一个 MSEventObj 对象。） 

[http://jsfiddle.net/aaronzhcl/Lz36b/](http://jsfiddle.net/aaronzhcl/Lz36b/)

# 参考
[HTML and DOM compatibility changes](http://msdn.microsoft.com/en-us/library/ie/dn467850%28v=vs.85%29.aspx)