---
layout: post
title:  "IE HTML DOM 兼容性变更"
date:   2014-05-08 22:49:54
categories: Compatibility
---

# 简介
随着Internet Explorer版本的更新，对HTML和DOM的功能有所更改，以便更好的支持现行标准及与其他浏览器行为保持一致。

IE针对HTML和DOM的主要功能更改如下，

+   [自动完成事件处理](#auto-complete)
+   [二进制行为和XML架构](#binary-behavior)
+   [内容属性和DOM属性](#content-attribute)
+   CreateElement方法
+   IFrame资源管理
+   MIME类型和纯文本内容
+   MIME类型和样式表
+   本机XML对象类型和传统网站
+   对象元素回退适用于DOM
+   克隆重叠元素
+   指针事件更新
+   减少MIME类型的安全风险
+   脚本元素和事件执行
+   表对象模型一致性
+   空格保存和DOM
+   Window事件行为已发生更改
+   当Window对象属性处于孤立状态时将被删除
+   XSLT兼容性更改 

<br/>

# 自动完成事件处理 <a name="auto-complete"></a>



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

# 内容属性和DOM属性

IE9内容属性(content-attribute)将不再连接到DOM expando, 这提高了Internet Explorer和其他浏览器之间的互操作性。

内容属性在是HTML源中指定的属性，例如，<element attribute1="value" attribute2="value">。许多内容属性都作为HTML的一部分进行预定义；HTML还支持用户自定义的内容属性。

在以下示例中，id和 class是HTML中预定义的内容属性，myAttr是用户定义的内容属性：
{% highlight html %}
<div id="myElement" class="b" myAttr="custom"></div>
{% endhighlight %}

在以下脚本示例中，id和 className是预定义的属性： 
{% highlight html %}
var div = document.getElementById("myElement");
var divId = div.id; // Gets the value of the id content attribute
var divClass = div.className; // Gets the value of the class content attribute
{% endhighlight %}

在IE8及以前版本中，包括IE8标准模式和IE9以前模式，存在myAttr内容属性表示存在myAttr DOM expando，
{% highlight html %}
var divExpando = div.myAttr; // divExpando would get the value "custom" in IE8
{% endhighlight %}

IE9中的重大更改是DOM expando不再由用户定义的内容属性的存在来表示，
{% highlight html %}
var divExpando = div.myAttr; // divExpando would get an undefined value
{% endhighlight %}

为解决此问题，请使用“getAttribute”API 检索用户定义的内容属性的值。建议所有版本的IE都使用此变通方法，并且新版本与旧版本IE相比不要求特殊外壳。 
{% highlight html %}
var divExpando = div.getAttribute("myAttr");
{% endhighlight %}

# 参考
[HTML and DOM compatibility changes](http://msdn.microsoft.com/en-us/library/ie/dn467850%28v=vs.85%29.aspx)