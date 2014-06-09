---
layout: post
title:  "IE JavaScript 兼容性变更"
date:   2014-05-07 22:49:54
categories: Compatibility
---


随着IE的新版本不断发布，对JavaScript语言的支持也相较之前版本有所变更，新版本中提高了对ECMAScript标准的支持，并于其他浏览器的JavaScript行为趋近一致。本文主要描述自IE9以来对JavaScript语言本身的主要变更。
<!--more-->

这些变更主要包括以下方面，

+   [数组索引处理](#array-index)
+   [枚举JavaScript属性](#enum-prop)
+   [函数指针方法调用](#function-pointer)
+   [夏令时处理](#daylight-saving)
+   [间接eval调用作用域](#eval-scope)
+   [数字精度和SSE2差异](#math-sse2)
+   [Null协议返回值](#null-return)
+   [验证JavaScript框架对新版IE的支持](#verify-js-framework)

<br/>
# 数组索引处理<a name="array-index"></a>

IE8在数组索引处理方面不符合 ECMAScript（第三方版本）规范。在创建索引大于2147483647的Array元素时，创建的新元素的索引将是一个负整数。

IE9正确处理了使用 2E+31-1 与 2E+32-2 之间的索引的 Array 元素。IE8行为不会在任何 IE9文档模式中复制。

*示例1*  
{% highlight javascript %}
function test() {
    var arr = new Array();		
    arr[2147483650] = 10000;
    arr.push(10);	
    document.write(arr["-2147483645"] == 10);
}
test();
{% endhighlight %}

*输出结果*  
IE8:
    "true"
IE9:
    "false"
 

# 枚举JavaScript属性<a name="enum-prop"></a>

IE9对JavaScript对象模型有所更改，使得JavaScript属性的枚举方式与较早版本中的枚举方式不同。

使用 for…in 语句时，属性枚举的顺序在任何文档模式中都可能与IE8返回的顺序不同。 

*示例1*  
IE9中数字属性会优先于非数字属性之前枚举。
{% highlight javascript %}
var obj = {first : "prop1", second: "prop2", 3: "prop3"};

var s = "";
for (var key in obj) {
    s += key + ": " + obj[key] + " ";
}
document.write (s);
{% endhighlight %}

*输出结果*  
IE8:
    first: prop1 second: prop2 3: prop3 

IE9:
    3: prop3 first: prop1 second: prop2

*示例2*  
IE8不包括与原型对象内置属性同名的属性的枚举。IE9中的所有文档模式在枚举中都包括这些属性。
{% highlight javascript %}
var obj = { first: "prop1", toString : "Hello" }
var s = "";
for (var key in obj) {
    s += key + ": " + obj[key] + " ";
}
document.write (s);
{% endhighlight %}

*输出结果*  
IE8:
    first: prop1

IE9:
    first: prop1 toString: Hello

# 函数指针方法调用<a name="function-pointer"></a>

IE9以前版本支持将方法的指针进行缓存并随后使用缓存的指针来调用方法。自IE9开始，取消了这项支持以改善与其他浏览器的互操作性。  
IE9需要有一个对象才能调用方法。默认情况下会作用于window对象。改进后有以下两种调用方式，

+   使用call方法（所有函数的一个属性）显式提供适当的调用对象
+   使用JavaScript的bind API将隐式调用对象与该方法关联

*示例1*

IE8中方法指针调用
{% highlight javascript %}
var d = document.writeln;
d("<script language=VBScript>");
{% endhighlight %}

*示例2*

IE9使用call方法显式指定对象
{% highlight javascript %}
d.call(document, "<script language="VBScript">”);
{% endhighlight %}

*示例3*

IE9使用bind方法隐式指定对象
{% highlight javascript %}
var d = document.writeln.bind(document);
d("<script language=VBScript>"); // Now this is OK.
{% endhighlight %}

# 夏令时处理 <a name="daylight-saving"></a>

IE9和以前版本中，日期通过应用ECMAScript规范中来存储夏令时调整时间。为提高准确性，尤其是过去日期（历史数据）的准确性，IE10依据系统规则存储夏令时调整时间。如果你的代码在你的Web应用程序中计算历史日期，或具有自定义逻辑可解决浏览器日期计算不准确的问题，要确保在升级Web应用程序使其适用于IE10时，自定义逻辑仍可正常使用。

对于夏令时转换发生在午夜（将时钟回拨）的时区，系统时间实际在过渡边界前1毫秒(ms)进行转换。通过在过渡边界前1ms进行转换，Windows 7及以上版本将仍处于夏令时转换的当天，但会在夏令时转换完成后的状态下向后回拨时钟。

*示例1*
{% highlight javascript %}
// Browser is running in Pacific Standard Time zone
new Date(Date.parse("3/31/2000")).toUTCString() 
{% endhighlight %}

*输出结果*  
IE10 (Standards mode): "Fri, 31 Mar 2000 07:00:00 UTC"

IE9 (Standards mode): "Fri, 31 Mar 2000 08:00:00 UTC"

*示例2*
{% highlight javascript %}
var milliSeconds = 0; 
var offSet1 = new Date(2012, 01, 25, 24, 00, 00, milliSeconds).getTimezoneOffset(); 
var offSet2 = new Date(2012, 01, 25, 24, 00, 00, milliSeconds-1).getTimezoneOffset(); // Check the offset 1 ms before
offSet1 != offSet2 ? alert("dstBoundary") : alert("non-dstBoundary");
{% endhighlight %}

*输出结果*  
IE10 (Standards mode): "dstBoundary"

IE9 (Standards mode): "non-dstBoundary"

# 间接eval调用作用域<a name="eval-scope"></a>

IE9以前版本，传递给间接eval的字符串将在本地函数作用域内求值。从IE9标准模式开始，该字符串根据ECMAScript语言规范第5版的规定在全局作用域中求值。 

*示例1*

{% highlight javascript %}
function test() {
   var dateFn = "Date(1971,3,8)";
   var myDate;
   var indirectEval = eval;
   indirectEval("myDate = new " + dateFn + ";");
   document.write(myDate);
}
test();
{% endhighlight %}

*输出结果*  
IE9 (Standards mode): "undefined"

IE8 : "Thu Apr 8 00:00:00 PDT 1971"

# 数字精度和SSE2差异<a name="math-sse2"></a>

IE9在平台支持的情况下会使用Streaming SIMD Extensions 2 (SSE2)来提高数学运算速度和精度，因此会获得和IE8及以前版本不同的精度。

*示例1*

{% highlight javascript %}
function test() {
    var x = 6.28318530717958620000;
    var val = Math.sin(x);
    document.write(Math.abs(val)) 
}
test();
{% endhighlight %}

*输出结果*  
IE9 (系统支持SSE2): "2.4492935982947064e-16"

IE8 : "2.4492127076447545e-16"

# Null协议返回值 <a name="null-return"></a>

IE9在处理返回"null"值的JavaScript时遵循以下HTML5规定。浏览器必须将URL视为已经返回HTTP 204 无内容，其中不得包含响应正文。

*示例1*

{% highlight html %}
<!DOCTYPE html>
<html>
<head>      
</head>
<body>
   <div id="ad_content">
      <iframe src="javascript:document.write('...'); return null;" />
   // document.write is meant to create the contents of the iframe
   </div>
</body>
</html>
{% endhighlight %}

由于作为JavaScript协议(javascript:)一部分执行返回"null"，IE9会将URL视为其返回了“HTTP 204 无内容”，因此iframe为空，无论JavaScript协议中运行其他什么JavaScript。

# 验证JavaScript框架对新版IE的支持<a name="verify-js-framework"></a>

许多站点仍在使用与新版本的IE不兼容的旧版JavaScript框架。许多现有JavaScript框架包含的功能取决于现有IE特定的行为或quirks模式。因此，在IE中所作的更改可能导致许多受欢迎的JavaScript框架部分无法正确工作。

为做演示，以下为需要更新以支持IE9的受欢迎 JavaScript 框架的列表。

+   Cufon 1.09i+
+   jQuery 1.5.1+
+   jQuery UI 1.6.8+
+   MooTools 1.3+
+   Prototype 1.7+

<br/>
# 参考文档

+   [JavaScript compatibility changes](http://msdn.microsoft.com/en-us/library/ie/dn467851%28v=vs.85%29.aspx)
+   [ECMAScript](http://www.ecmascript.org/docs.php)