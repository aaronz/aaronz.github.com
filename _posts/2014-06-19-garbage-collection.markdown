---
layout: post
title:  "浏览器垃圾回收"
date:   2014-06-19 00:00:00
categories: Mechanism
---

浏览器的可使用内存数量通常要比分配给桌面应用程序的少。这样主要为防止运行JavaScript的网页耗尽全部系统内存而导致系统崩溃。浏览器在执行JavaScript语言是要负责脚本运行过程中的内存管理，脚本的编写者编程中只需要关心调用构造函数来创建对象，但不需要处理内存的回收，这是因为垃圾回收机制在默默的工作，帮我们把不用的内存回收重用。

![Garbage Collection](/assets/images/posts/walle.jpg)

<!--more-->

# 原理

垃圾回收机制原理其实很简单，即找到不再被使用的对象，释放其内存。然后按照一定的触发条件周期性的触发这个过程。垃圾回收最重要的一点就是确定如何确定垃圾对象，现实的实现中主要有两种机制，

+   标记清除 – 由根对象开始标记可到达对象，然后对不可达对象进行回收。
+   引用计数 – 维护对象的引用计数，实现上很难避免循环引用带来的困扰。

## 标记清除

1.  当变量进入环境（例如，声明变量）时，这个变量标记为“进入环境”。当变量离开环境时，这将其 标记为“离开环境”。
2.  垃圾收集器在运行的时候会给存储在内存中的所有变量都加上标记。
3.  去掉环境中变量以及被环境中的变量引用的变量标记。而在此之后仍带有标记的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。
4.  垃圾收集器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。

IE，Firefox，Opera，Chrome和Safari的目前都是使用的标记清除回收策略。

## 引用计数

1.  跟踪记录每个值被引用的次数。当声明一个变量并将引用类型的值赋给该变量时，则这个值的引用次数就是1。
2.  如果同一个值又被赋给另一个变量，则该值的引用次数加1。相反，如果包含对这个值引用的变量又取得另外一个值，则这个值的引用次数减1.
3.  当这个值的引用次数变成0时，则说明没有办法访 问这个值了，因此就可以将其占用的内存空间回收回来。这样当垃圾收集器下次再运行时，它就会释放那些引用次数为零的值所占用的内存

## 引用计数的缺陷

引用计数带来一个严重的问题是循环引用，例如下面例子,

{% highlight javascript%}
<script>
    function () {
        var objectA = new Object();
        var objectB = new Object();
        objectA.someOtherObject = objectB;
        objectB.anotherObject = objectA;
    }
</script>
{% endhighlight %}

在这个例子中，objectA和objectB通过各自的属性相互引用，也就是说，这两个对象的引用次数都是2。在采用引标记清除略的实现中，由于函数执 行之后，这两个对象都离开了作用域。因此这两种相互引用不是个问题。但在采用引用计数策略的实现中，但函数执行完毕后，objectA和objectB还 将继续存在，因此他们的引用次数永远不会是0。假如这个函数被重复调用，就会导致大量的内存得不到回收。因此，Netscape在Navigator 4.0中放弃了引用计数器方式，转而采用标记清除来实现对其垃圾回收机制。

可是，引用计数导致的麻烦并未就此终结。

IE中有一部分对象并不是原生JavaScript对象。其中BOM和DOM中的对象就是使用C++以COM 对象的形式实现的，而COM对象的垃圾收集机制采用的就是引用计数策略。因此，即使IE的JavaScript引擎是使用标记清除策略来实现的，但JavaScript访问的COM对象依然是基于引用计数策略的。换句话说，只要IE中涉及COM对象，就会存在循环引用的问题。

{% highlight javascript%}

var element = document.getElementById("some_element");
var myObject = new Object();
myObject.element = element;
element.somObject = myObject;

// Remove the circular reference
myObject.element = null;
element.somObject = null;
{% endhighlight %}

这里例子在一个DOM元素(element)与一个原生的javascript对象(myObject)之间创建了循环引用。其中，变量myObject 有一个名为element的属性指向element对象；而变量element也有一个属性名叫someObject回指myObject。由于存在这个 循环引用，即使将例子中的DOM从页面中移除，它也永远不会被回收。
将变量设置为null，意味着切断变量与它此前引用的值之间的连接。但垃圾收集器下次运行时，就会删除这些值并回收它们占用的内存。

# IE的垃圾回收

IE6垃圾回收 - 根据内存分配量运行的，具体一点说就是256个变量、4096个对象（或数组）和数组元素（slot）或者64KB的字符串。达到上述任何一个临界值，垃圾收集器就会运行。问题在于如果一个脚本中包含那么多 变量，那么该脚本很可能会在其生命中起一直保持那么多的变量，垃圾收集器就可能不得不频繁的运行。

IE7垃圾回收 - IE7中的各项临界值在初始化时与IE6相等。如果例程回收的内存分配量低于15%，则变量 、字面量和（或）数组元素的临界值就会加倍。如果例程回收了85%的内存分配量，则将各种临界重置会默认值。这一看似简单的调整，极大地提升了IE在运行 包含大量JavaScript的页面时的性能。

强制垃圾回收 - 有的浏览器中可以触发垃圾收集过程，在IE中，调用window.CollectGarbage()方法会立即指向垃圾收集，在Opera7及更高版本中，调用widnow.opera.collect()也会启动垃圾收集例程。

# 内存泄漏

由于IE对JScript对象和COM对象使用不同的垃圾收集例程，因此闭包在IE中会导致一些特殊的问题。具体来说，如果闭包的作用域链中保存着一个HTML元素，那么就意味着该元素无法被销毁。

{% highlight javascript%}
function assignHandler() {
        var element = document.getElementById("someElement");
        element.onclick = function () {
            alert(element.id);
        };
        element = null;
    };
{% endhighlight %}

以上代码创建了一个作为element元素时间处理程序的闭包，而这个闭包则有创建了一个循环引用。由于匿名函数保存了一个对 assignHandler()的活动对象的引用，因此就会导致无法减少element的引用数。只要匿名函数存在，element的引用数至少也是1，因此它所占用的内存就永远不会被回收。

不过，这个问题可以通过稍微改写一下代码来解决。在上面代码中，通过把element.id的一个副本保存在一个变量中，并且在闭包中引用该变量消除了循环引用。但仅仅做到这一步，还是不能解决内存泄漏的问题。必须要记住：闭包会引用包含函数活动的整个活动对象，而其中包含着element。即使闭包不直接引用element，包含函数的活动对象中也仍 然会保存一个引用。因此，有必要把element变量设置为null。这样就能够解除对DOM对象的引用，顺利地减少其引用数，确保正常回收其占用的内存。

# 参考

[Professional JavaScript for Web Developers](http://www.amazon.com/Professional-JavaScript-Developers-Nicholas-Zakas/dp/1118026691/) - Nicholas C. Zakas