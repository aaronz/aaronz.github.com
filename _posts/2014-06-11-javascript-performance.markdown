---
layout: post
title:  "JavaScript性能优化指南"
date:   2014-06-11 00:00:00
categories: Performance
---

如今主流浏览器都在比拼JavaScript引擎的执行速度，但最终都会达到一个理论极限，即无限接近编译后程序执行速度。
这种情况下决定程序速度的另一个重要因素就是代码本身。

在这里我们会分门别类的介绍JavaScript性能优化的技巧，并提供相应的测试用例，供大家在自己使用的浏览器上验证，
同时会对特定的JavaScript背景知识做一定的介绍。

<!--more-->


<div class="post-index">
<ul>
<li><a href="#variable-lookup">变量查找优化</a>
<ul>
<li><a href="#declare-with-var">变量声明带上var</a></li>
<li><a href="#global-with-patient">慎用全局变量</a></li>
<li><a href="#cache-global-variable">缓存重复使用的全局变量</a></li>
<li><a href="#avoid-with">避免使用with</a></li>
</ul>
</li>
<li><a href="#core">核心语法优化</a>
<ul>
<li><a href="#prototype">通过原型优化方法定义</a></li>
<li><a href="#closure">避开闭包陷阱</a></li>
<li><a href="#accessor">避免使用属性访问方法</a></li>
<li><a href="#try-catch">避免在循环中使用try-catch</a></li>
<li><a href="#for-in">使用for代替for&hellip;in&hellip;遍历数组</a></li>
<li><a href="#native-operator">使用原始操作代替方法调用</a></li>
<li><a href="#string-function">传递方法取代方法字符串</a></li>
</ul>
</li>
<li><a href="#script-loading">脚本装载优化</a>
<ul>
<li><a href="#minify">使用工具精简脚本</a></li>
<li><a href="#gzip">启用Gzip压缩</a></li>
<li><a href="#cache-control">设置Cache-Control和Expires头</a></li>
<li><a href="#async-load">异步加载脚本</a></li>
</ul>
</li>
<li><a href="#dom-manipulation">DOM操作优化</a>
<ul>
<li><a href="#reduce-dom">减少DOM元素数量</a></li>
<li><a href="#css-change">优化CSS样式转换</a></li>
<li><a href="#node-adding">优化节点添加</a></li>
<li><a href="#node-modification">优化节点修改</a></li>
<li><a href="#position-property">减少使用元素位置操作</a></li>
<li><a href="#loop-dom">避免遍历大量元素</a></li>
</ul>
</li>
<li><a href="#events">事件优化</a>
<ul>
<li><a href="#event-delegate">使用事件代理</a></li>
</ul>
</li>
<li><a href="#animation">动画优化</a>
<ul>
<li><a href="#position">设置动画元素为absolute或fixed</a></li>
<li><a href="#timer">使用一个timer完成多个元素动画</a></li>
</ul>
</li>
</ul>
</div>


变量查找优化 <a name="variable-lookup"></a>
============

变量声明带上var <a name="declare-with-var"></a>
---------------

1.
如果声明变量忘记了var，那么js引擎将会遍历整个作用域查找这个变量，结果不管找到与否，都是悲剧。

-   如果在上级作用域找到了这个变量，上级作用域变量的内容将被无声的改写，导致莫名奇妙的错误发生。
-   如果在上级作用域没有找到该变量，这个变量将自动被声明为全局变量，然而却都找不到这个全局变量的定义。

2\. 基于上面逻辑，性能方面不带var声明变量自然要比带var速度慢

具体可以参考[http://jsperf.com/withvar-withoutvar](http://jsperf.com/withvar-withoutvar/3 "http://jsperf.com/withvar-withoutvar/3")。下面是个简单的结果截图，蓝色为带var的情况，越长说明
速度越快。

[![image](http://images.cnitblog.com/blog/502305/201312/25231302-eaf1e449591143b0bbd5907fcb4aacf2.png "image")](http://images.cnitblog.com/blog/502305/201312/25231302-4ff08ee2f1634956a35c6eb605a3a1b1.png)

慎用全局变量 <a name="global-with-patient"></a>
------------

1\. 全局变量需要搜索更长的作用域链。

2\. 全局变量的生命周期比局部变量长，不利于内存释放。

3\. 过多的全局变量容易造成混淆，增大产生bug的可能性。

全局变量与局部变量的测试可以参考[http://](http://jsperf.com/local-global-var/3)[jsperf.com/local-global-var](http://jsperf.com/local-global-var/3)

 

以上两条还可以得出一条JavaScript**常用的编程风格**，**具有相同作用域变量通过一个var声明
。**

这样方便查看该作用域所有的变量，JQuery源代码中就是用了这种风格。例如下面源代码

<https://github.com/jquery/jquery/blob/master/src/core.js>

{% highlight html %}
jQuery.extend = jQuery.fn.extend = function() {
var options, name, src, copy, copyIsArray, clone,target = arguments[0] || {},i = 1,length = 

arguments.length,deep = false;
{% endhighlight %}

缓存重复使用的全局变量 <a name="cache-global-variable"></a>
----------------------

1\. 全局变量要比局部变量需要搜索的作用域长

2\. 重复调用的方法也可以通过局部缓存来提速

3\. 该项优化在IE上体现比较明显

缓存与不缓存变量的测试可以参考<http://jsperf.com/localvarcache>

JQuery源代码中也是用了类似的方法，<https://github.com/jquery/jquery/blob/master/src/selector-native.js>

{% highlight javascript %}
var docElem = window.document.documentElement, selector_hasDuplicate,
matches = docElem.webkitMatchesSelector || docElem.mozMatchesSelector || docElem.oMatchesSelector || 

docElem.msMatchesSelector,
selector_sortOrder = function( a, b ) {
// Flag for duplicate removal
if ( a === b ) {
    selector_hasDuplicate = true;
    return 0;
}
{% endhighlight %}

避免使用with <a name="avoid-with"></a>
------------

with语句将一个新的可变对象推入作用域链的头部，函数的所有局部变量现在处于第二个作用域链对象中，从而使局部变
量的访问代价提高。

{% highlight javascript %}
var person = {
    name: “Nicholas",
    age: 30
}
function displayInfo() {
    var count = 5;
    with (person) {
        alert(name + ' is ' + age);
        alert('count is ' + count);
    }
}
{% endhighlight %}

以上代码的结果将name和age两个变量推入第一个作用域，如下图所示，

[![image](http://images.cnitblog.com/blog/502305/201312/25230614-52d6ee6766b44684a479085fe553e2d5.png "image")](http://images.cnitblog.com/blog/502305/201312/25230612-35147949ff31422dab627a30ea8af14e.png)

使用with与不使用with的测试可以参考[http://](http://jsperf.com/with-with)[jsperf.com/with-with](http://jsperf.com/with-with/2)

核心语法优化 <a name="core"></a>
============

通过原型优化方法定义 <a name="prototype"></a>
--------------------

1.
如果一个方法类型将被频繁构造，通过方法原型从外面定义附加方法，从而避免方法的重复定义。\
2. 可以通过外
部原型的构造方式初始化**值类型**的变量定义。（这里强调值类型的原因是，引用类型如果在原型中定义，
一个实例对引用类型的更改会影响到其他实例。）

这条规则中涉及到JavaScript中原型的概念，

-   构造函数都有一个prototype属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。我们可
    以把那些不变的属性和方法，直接定义在prototype对象上。
-   可以通过对象实例访问保存在原型中的值，不能通过对象实例重写原型中的值。
-   在实例中添加一个与实例原型同名属性，那该属性就会屏蔽原型中的属性。
-   通过delete操作符可以删除实例中的属性。

例如以下代码以及相应的内存中原型表示如下，

{% highlight javascript %}
function Person(){}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
    alert(this.name);
};
var person1 = new Person();
person1.sayName(); //”Nicholas”
var person2 = new Person();
person2.sayName(); //”Nicholas”
{% endhighlight %}

[![image](http://images.cnitblog.com/blog/502305/201312/25233912-364f6e163bd44522823042a232167846.png "image")](http://images.cnitblog.com/blog/502305/201312/25233911-83ab3f38f8484c72a2ac802f2827237d.png)

原型附加方法测试可以参考[http://](http://jsperf.com/func-constructor)[jsperf.com/func-constructor](http://jsperf.com/func-constructor)

原型附加值类型变量测试可以参考[http://](http://jsperf.com/prototype2)[jsperf.com/prototype2](http://jsperf.com/prototype2)

避开闭包陷阱 <a name="closure"></a>
------------

1.
闭包是个强大的工具，但同时也是性能问题的主要诱因之一。不合理的使用闭包会导致内存泄漏。

2\. 闭包的性能不如使用内部方法，更不如重用外部方法。

由于IE浏览器的DOM是用COM来实现的，
COM的内存管理是通过引用计数的方式，引用计数有个难题就是循环引用，一旦DOM
引用了闭包(例如event
handler)，闭包的上层元素又引用了这个DOM，就会造成循环引用从而导致内存泄漏。

![Figure 2 Circular References with
Closures](http://i.msdn.microsoft.com/dynimg/IC133807.gif "Figure 2 Circular References with Closures")

关于Js内存泄漏可以参考

<http://www.crockford.com/javascript/memory/leak.html>

<http://msdn.microsoft.com/en-us/library/bb250448%28v=vs.85%29.aspx>

闭包与非闭包的测试<http://jsperf.com/closure2>

避免使用属性访问方法 <a name="accessor"></a>
--------------------

1\. JavaScript不需要属性访问方法，因为所有的属性都是外部可见的。\
2. 添加属性访问方法只是增加了一层重定向 ，对于访问控制没有意义。

使用属性访问方法示例

{% highlight javascript %}
function Car() {     
  this.m_tireSize = 17;     
  this.m_maxSpeed = 250;
  this.GetTireSize = Car_get_tireSize;     
  this.SetTireSize = Car_put_tireSize;
}

function Car_get_tireSize() {     
  return this.m_tireSize;
}

function Car_put_tireSize(value) {     
  this.m_tireSize = value;
}
var ooCar = new Car();
var iTireSize = ooCar.GetTireSize();
ooCar.SetTireSize(iTireSize + 1);
{% endhighlight %}

直接访问属性示例

{% highlight javascript %}
function Car() {     
  this.m_tireSize = 17;     
  this.m_maxSpeed = 250;
}
var perfCar = new Car();
var iTireSize = perfCar.m_tireSize;
perfCar.m_tireSize = iTireSize + 1;
{% endhighlight %}

使用属性访问与不使用属性访问的测试[http://](http://jsperf.com/property-accessor)[jsperf.com/property-accessor](http://jsperf.com/property-accessor)

避免在循环中使用try-catch <a name="try-catch"></a>
-------------------------

1.
try-catch-finally语句在catch语句被执行的过程中会动态构造变量插入到当前域中，对性能有一定影响。\
2. 如 果需要异常处理机制，可以将其放在循环外层使用。

循环中使用try-catch

{% highlight javascript %}
for (var i = 0; i < 200; i++) {
 try {} catch (e) {}
}
{% endhighlight %}

循环外使用try-catch

{% highlight javascript %}
try {
 for (var i = 0; i < 200; i++) {}
} catch (e) {}
{% endhighlight %}

循环内与循环外使用try-catch的测试<http://jsperf.com/try-catch>

使用for代替for…in…遍历数组 <a name="for-in"></a>
--------------------------

for…in…内部实现是构造一个所有元素的列表，包括array继承的属性，然后再开始循环。相对for循环性能要慢。

StackOverflow上对这个for和for
in的问题有个[经典的回答](http://stackoverflow.com/questions/500504/why-is-using-for-in-with-array-iteration-such-a-bad-idea)，直接原文引用，

Q: I've been told not to use "for...in" with arrays in JavaScript. Why
not?

A: The reason is that one construct...

{% highlight javascript %}
var a = [];
a[5] = 5; // Perfectly legal JavaScript that resizes the array.

for (var i=0; i<a.length; i++) {
    // Iterates over numeric indexes from 0 to 5, as everyone expects.
}
{% endhighlight %}

can sometimes be totally different from the other...

{% highlight javascript %}
var a = [];
a[5] = 5;
for (var x in a) {
    // Shows only the explicitly set index of "5", and ignores 0-4
}
{% endhighlight %}

Also consider that [JavaScript](http://en.wikipedia.org/wiki/JavaScript)
libraries might do things like this, which will affect any array you
create:

{% highlight javascript %}
// Somewhere deep in 

your JavaScript library...
Array.prototype.foo = 1;

// Now you have no idea what the below code will do.
var a = [1,2,3,4,5];
for (var x in a){
    // Now foo is a part of EVERY array and 
    // will show up here as a value of 'x'.
}
{% endhighlight %}

关于for和for…in…的测试可以看[http://](http://jsperf.com/forin/6)[jsperf.com/forin](http://jsperf.com/forin/6)

使用原始操作代替方法调用 <a name="native-operator"></a>
------------------------

方法调用一般封装了原始操作，在性能要求高的逻辑中，可以使用原始操作代替方法调用来提高性能。

原始操作

{% highlight javascript %}
var min = a < b ? a : b;
{% endhighlight %}

方法实例

{% highlight javascript %}
var min = Math.min(a, b);
{% endhighlight %}

关于方法调用和原始操作的测试参考[http://](http://jsperf.com/operator-function)[jsperf.com/operator-function](http://jsperf.com/operator-function)

传递方法取代方法字符串 <a name="string-function"></a>
----------------------

一些方法例如setTimeout()/setInterval()，接受字符串或者方法实例作为参数。直接传递方法对象作为参数来避免对字
符串的二次解析。

传递方法

{% highlight javascript %}
setTimeout(test, 1);
{% endhighlight %}

传递方法字符串

{% highlight javascript %}
setTimeout('test()', 1);
{% endhighlight %}

对应的测试可以参考[http://](http://jsperf.com/string-function)[jsperf.com/string-function](http://jsperf.com/string-function)

脚本装载优化 <a name="script-loading"></a>
============

使用工具精简脚本 <a name="minify"></a>
----------------

精简代码就是将代码中的空格和注释去除，也有更进一步的会对变量名称混淆+精简。

根据统计精简后文件大小平均减少21%，即使Gzip之后文件也会减少5%。

常用的工具如下，

-   [JSMin](http://crockford.com/javascript/jsmin)
-   [Closure compiler](http://code.google.com/intl/pl/closure/compiler/)
-   [YUICompressor](http://developer.yahoo.com/yui/compressor/)

例如Closure Compiler效果如下，

[![image](http://images.cnitblog.com/blog/502305/201312/27133908-b5ac066e8990485abd52ed3bb19ae451.png "image")](http://images.cnitblog.com/blog/502305/201312/27133907-dad743a830394666a821b63f1881c99e.png)

启用Gzip压缩 <a name="gzip"></a>
------------

Gzip通常可以减少70%网页内容的大小，包括脚本、样式表、图片等文件。Gzip比deflate更高效，主流服务器都有相应的
压缩支持模块。

Gzip的工作流程为

-   客户端在请求Accept-Encoding中声明可以支持gzip
-   服务器将请求文档压缩，并在Content-Encoding中声明该回复为gzip格式
-   客户端收到之后按照gzip解压缩

[![image](http://images.cnitblog.com/blog/502305/201312/27133911-8eb3f06426774f19a23b136d318228a0.png "image")](http://images.cnitblog.com/blog/502305/201312/27133909-66adcdb1913541d7a05f9917f16f9178.png)

设置Cache-Control和Expires头 <a name="cache-control"></a>
----------------------------

通过Cache-Control和Expires头可以将脚本文件缓存在客户端或者代理服务器上，可以减少脚本下载的时间。

{% highlight html %}
Expires格式:
Expires = "Expires" ":" HTTP-date
Expires: Thu, 01 Dec 1994 16:00:00 GMT
Note: if a response includes a Cache-Control field with the max-age directive that directive overrides the
Expires field.

Cache-Control格式：
Cache-Control   = "Cache-Control" ":" 1#cache-directive
Cache-Control: public
{% endhighlight %}

具体的标准定义可以参考http1.1中的定义，简单来说Expires控制过期时间是多久，Cache-Control控制什么地方可以缓存
。

<http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21>

<http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9>

异步加载脚本 <a name="async-load"></a>
------------

脚本加载与解析会阻塞HTML渲染，可以通过异步加载方式来避免渲染阻塞。

异步加载的方式很多，比较通用的方法是通过类似下面的代码实现，

{% highlight javascript %}
function loadjs

(script_filename){
    var script = document.createElement('script');
    script.setAttribute('type', 'text/javascript');
    script.setAttribute('src', script_filename);
    script.setAttribute('id', 'script-id');

    scriptElement = document.getElementById('script-id');
    if(scriptElement){
        document.getElementsByTagName('head')[0].removeChild(scriptElement);
    }
    document.getElementsByTagName('head')[0].appendChild(script);
}
var script = 'scripts/alert.js';
loadjs(script);
{% endhighlight %}

DOM操作优化 <a name="dom-manipulation"></a>
===========

DOM操作性能问题主要有以下原因，

-   DOM元素过多导致元素定位缓慢
-   大量的DOM接口调用
-   DOM操作触发频繁的reflow(layout)和repaint

关于reflow(layout)和repaint可以参考下图，可以看到layout发生在repaint之前，所以layout相对来说会造成更多性能
损耗。

-   reflow(layout)就是计算页面元素的几何信息
-   repaint就是绘制页面元素

[![image](http://images.cnitblog.com/blog/502305/201312/27141557-a4f1a931a3bd49d4af67e38d577bddb3.png "image")](http://images.cnitblog.com/blog/502305/201312/27141544-294a7fc900d54879869f6ccd0938b945.png)

以下是一个wikipedia网站reflow的过程录像，
<p><object width="480" height="400" align="middle" data="http://player.youku.com/player.php/sid/XMzI5MDg0OTA0/v.swf" type="application/x-shockwave-flash"><param name="src" value="http://player.youku.com/player.php/sid/XMzI5MDg0OTA0/v.swf" /><param name="allowfullscreen" value="true" /><param name="quality" value="high" /><param name="allowscriptaccess" value="always" /></object></p>
 

减少DOM元素数量 <a name="reduce-dom"></a>
---------------

1\. 在console中执行命令查看DOM元素数量

{% highlight javascript %}
    document.getElementsByTagName('*').length 
{% endhighlight %}

2\. Yahoo首页DOM元素数量在1200左右。正常页面大小一般不应该超过 1000。\
3.
DOM元素过多会使DOM元素查询效率，样式表匹配效率降低，是页面性能最主要的瓶颈之一。

优化CSS样式转换 <a name="css-change"></a>
---------------

如果需要动态更改CSS样式，尽量采用触发reflow次数较少的方式。

例如以下代码逐条更改元素的几何属性，理论上会触发多次reflow

{% highlight javascript %}
element.style.fontWeight = 'bold';
element.style.marginLeft= '30px';
element.style.marginRight = '30px';
{% endhighlight %}

可以通过直接设置元素的className直接设置，只会触发一次reflow

{% highlight javascript %}
element.className = 

'selectedAnchor';
{% endhighlight %}

具体的测试结果如下，

[![image](http://images.cnitblog.com/blog/502305/201312/27142905-c7553f9b871b4f79ae965faa011da211.png "image")](http://images.cnitblog.com/blog/502305/201312/27142903-d3b9897c07694ea5b44d58a7fd80a780.png)

测试用例可以参考[http://jsperf.com/css-class](http://jsperf.com/css-class/2 "http://jsperf.com/css-class/2")

优化节点添加 <a name="node-adding"></a>
------------

多个节点插入操作，即使在外面设置节点的元素和风格再插入，由于多个节点还是会引发多次reflow。优化的方法是创建
DocumentFragment，在其中插入节点后再添加到页面。

例如JQuery中所有的添加节点的操作如append，都是最终调用documentFragment来实现的，

<http://code.jquery.com/jquery-1.10.2.js>

{% highlight javascript %}
function 

createSafeFragment( document ) {
    var list = nodeNames.split( "|" ),
        safeFrag = document.createDocumentFragment();

    if ( safeFrag.createElement ) {
        while ( list.length ) {
            safeFrag.createElement(
                list.pop()
            );
        }
    }
    return safeFrag;
}
{% endhighlight %}

关于documentFragment对比直接添加节点的测试<http://jsperf.com/fragment2>

优化节点修改 <a name="node-modification"></a>
------------

对于节点的修改，可以考虑使用cloneNode在外部更新节点然后再通过replace与原始节点互换。

{% highlight javascript %}
var orig = document.getElementById('container');
var clone = orig.cloneNode(true);
var list = ['foo', 'bar', 'baz'];
var contents;
for (var i = 0; i < list.length; i++) {
  content = document.createTextNode(list[i]);
  clone.appendChild(content);
}
orig.parentNode.replaceChild(clone, orig);
{% endhighlight %}

对应的测试可以参考<http://jsperf.com/clone-node2>

减少使用元素位置操作 <a name="position-property"></a>
--------------------

一般浏览器都会使用增量reflow的方式将需要reflow的操作积累到一定程度然后再一起触发，但是如果脚本中要获取以下
属性，那么积累的reflow将会马上执行，已得到准确的位置信息。

-   offsetLeft
-   offsetTop
-   offsetHeight
-   offsetWidth
-   scrollTop/Left/Width/Height
-   clientTop/Left/Width/Height
-   getComputedStyle()

具体讨论可以参考这个链接[http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-
performance-making-your-javascript-slow/\#comment-13157](http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/#comment-13157 "http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-

performance-making-your-javascript-slow/#comment-13157")

避免遍历大量元素 <a name="loop-dom"></a>
----------------

避免对全局DOM元素进行遍历，如果parent已知可以指定parent在特定范围查询。

例如以下示例，

{% highlight javascript %}
var elements = document.getElementsByTagName('*');
for (i = 0; i < elements.length; i++) {
  if (elements[i].hasAttribute('selected')) {}
}
{% endhighlight %}

如果已知元素存在于一个较小的范围内，

{% highlight javascript %}
var elements = document.getElementById

('canvas').getElementsByTagName('*');
for (i = 0; i < elements.length; i++) {
  if (elements[i].hasAttribute('selected')) {}
}
{% endhighlight %}


相关测试可以参考<http://jsperf.com/ranged-loop>

事件优化 <a name="events"></a>
========

使用事件代理 <a name="event-delegate"></a>
------------

1.
当存在多个元素需要注册事件时，在每个元素上绑定事件本身就会对性能有一定损耗。\
2. 由于DOM Level2事件模
型中所有事件默认会传播到上层文档对象，可以借助这个机制在上层元素注册一个统一事件对不同子元素进行相应处理。

捕获型事件先发生。两种事件流会触发DOM中的所有对象，从document对象开始，也在document对象结束。

[http://](http://www.w3.org/TR/2003/NOTE-DOM-Level-3-Events-20031107/events.html)[www.w3.org/TR/2003/NOTE-DOM-Level-3-Events-20031107/events.html](http://www.w3.org/TR/2003/NOTE-DOM-Level-3-Events-20031107/events.html)

[![image](http://images.cnitblog.com/blog/502305/201312/27152552-192d52c545b34ae0bb6db90bb82a6ba4.png "image")](http://images.cnitblog.com/blog/502305/201312/27152551-c0b17e5640f5449e8aa8ddbbb1f1446f.png)

示例代码如下

{% highlight html %}
<ul id="parent-list">
    <li id="post-1">Item 1
    <li id="post-2">Item 2
    <li id="post-3">Item 3
    <li id="post-4">Item 4
    <li id="post-5">Item 5
    <li id="post-6">Item 6
</li></ul>
{% endhighlight %}

{% highlight html %}
// Get the element, add a click listener...
document.getElementById("parent-list").addEventListener("click",function(e) {
    // e.target is the clicked element!
    // If it was a list item
    if(e.target && e.target.nodeName == "LI") {
        // List item found!  Output the ID!
        console.log("List item ",e.target.id.replace("post-")," was clicked!");
    }
});
{% endhighlight %}

对应的测试可以参考[http://jsperf.com/event-
delegate](http://jsperf.com/event-delegate)

动画优化 <a name="animation"></a>
========

动画效果在缺少硬件加速支持的情况下反应缓慢，例如手机客户端

特效应该只在确实能改善用户体验时才使用，而不应用于炫耀或者弥补功能与可用性上的缺陷

至少要给用户一个选择可以禁用动画效果

设置动画元素为absolute或fixed <a name="position"></a>
-----------------------------

position: static 或position: relative元素应用动画效果会造成频繁的reflow

position: absolute或position: fixed 的元素应用动画效果只需要repaint

关于position的具体介绍可以参考

[http://css-
tricks.com/almanac/properties/p/position](http://css-tricks.com/almanac/properties/p/position/)[/](http://css-tricks.com/almanac/properties/p/position/)

使用一个timer完成多个元素动画 <a name="timer"></a>
-----------------------------

setInterval和setTimeout是两个常用的实现动画的接口，用以间隔更新元素的风格与布局。

动画效果的帧率最优化的情况是使用一个timer完成多个对象的动画效果，其原因在于多个timer的调用本身就会损耗一定
性能。

{% highlight javascript %}
setInterval(function() {
  animateFirst('');
}, 10);
setInterval(function() {
  animateSecond('');
}, 10);
{% endhighlight %}

使用同一个timer，

{% highlight javascript %}
setInterval(function() {
  animateFirst('');
  animateSecond('');
}, 10);
{% endhighlight %}

 

以上是JavaScript性能提高的技巧总结，基本上都能够通过测试验证，但是限于篇幅没有把所有的测试结果都
贴出来。

最后再引用一句名人名言作为结尾，

Premature optimization is the root of all evil.                  --
Donald Knuth

 

 


