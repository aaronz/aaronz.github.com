---
layout: post
title:  "IE JavaScript 兼容性变更"
date:   2014-05-07 22:49:54
categories: 兼容性
---
# 简介
随着Internet Explorer的新版本不断发布，对JavaScript语言的支持也相较之前版本有所变更，新版本中提高了对ECMAScript标准的支持，并于其他浏览器的JavaScript行为趋近一致。本文主要描述自Internet Explorer 9以来对JavaScript语言本身的主要变更。

这些变更主要包括以下方面，

+   [数组索引处理](#array-index)
+   [枚举JavaScript属性](#enum-prop)
+   函数指针方法调用
+   JavaScript夏令时处理
+   间接eval调用作用域
+   数字精度和SSE2差异
+   Null协议返回值
+   验证JavaScript框架对新版IE的支持  

  

# 数组索引处理<a name="array-index"></a>

Internet Explorer 8 在数组索引处理方面不符合 ECMAScript（第三方版本）规范。在创建索引大于 2147483647 的 Array 元素时，创建的新元素的索引将是一个负整数。

Internet Explorer 9 正确处理了使用 2E+31-1 与 2E+32-2 之间的索引的 Array 元素。Internet Explorer 8 行为不会在任何 Internet Explorer 9 文档模式中复制。

**示例1**
{% highlight javascript %}
function test() {
    var arr = new Array();		
    arr[2147483650] = 10000;
    arr.push(10);	
    document.write(arr["-2147483645"] == 10);
}
test();
{% endhighlight %}

**输出结果**
Internet Explorer 8

    "true"
Internet Explorer 9 

    "false"
 

# 枚举JavaScript属性<a name="enum-prop"></a>

Internet Explorer 9 对JavaScript对象模型有所更改，使得JavaScript属性的枚举方式与较早版本中的枚举方式不同。

使用 for…in 语句时，属性枚举的顺序在任何文档模式中都可能与Internet Explorer 8返回的顺序不同。 

**示例1**

Internet Explorer 9 中数字属性会优先于非数字属性之前枚举。
{% highlight javascript %}
var obj = {first : "prop1", second: "prop2", 3: "prop3"};

var s = "";
for (var key in obj) {
    s += key + ": " + obj[key] + " ";
}
document.write (s);
{% endhighlight %}

**输出结果**

All modes of Internet Explorer 8:

    first: prop1 second: prop2 3: prop3 

All modes of Internet Explorer 9:

    3: prop3 first: prop1 second: prop2

**示例2**

Internet Explorer 8 不包括与原型对象内置属性同名的属性的枚举。Internet Explorer 9 中的所有文档模式在枚举中都包括这些属性。
{% highlight javascript %}
var obj = { first: "prop1", toString : "Hello" }
var s = "";
for (var key in obj) {
    s += key + ": " + obj[key] + " ";
}
document.write (s);
{% endhighlight %}

**输出结果**

All modes of Internet Explorer 8:

    first: prop1

All modes of Internet Explorer 9:

    first: prop1 toString: Hello

