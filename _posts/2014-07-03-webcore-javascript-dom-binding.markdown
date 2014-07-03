---
layout: post
title:  "WebKit JavaScript DOM对象绑定"
date:   2014-07-03 00:00:00
categories: Mechanism
---

浏览器中的各种精彩的动态效果大多数都是通过JavaScript调用DOM对象来完成的，但是浏览器是如何实现JavaScript脚本对浏览器的DOM对象访问的？本文通过解析WekKit实现来揭开答案。

![webkit](/assets/images/posts/webkit.jpg)

<!--more-->

# Web IDL

WebCore作为一个模块化的浏览器引擎，如何才能很容易的与其他组件集成？例如ObjectC，JavaScriptCore组件等。答案是通过Web IDL。Web IDL是一种标记语言，通过它可以定义对外暴露的接口。当WebKit编译时会通过脚本根据Web IDL定义生成对应的绑定源代码，从而将WebCore对象绑定到外界组件。

**注意**: [WebCore采用的Web IDL](https://trac.webkit.org/wiki/WebKitIDL)与[标准的Web IDL](http://www.w3.org/TR/WebIDL/)存在区别。例如Web IDL标准中定义方法的关键字为operation而不是method。标准Web IDL不区分attribute与parameter。

下面是一个Web IDL的简单实例。

+ Node被定义为一个interface。
+ ELEMENT_NODE是Node中的一个常量。
+ parentNode和nodeName是Node interface的属性。
+ appendChild和addEventListener是Node interface中的方法。
+ type，lintener和useCapture是addEventListener方法的参数。
+ [CustomToJSObject], [TreatReturnedNullStringAs=Null], [Custom] and [CustomReturn]是IDL的控制属性。

{% highlight text %}

module core {
    [
        CustomToJSObject
    ] interface Node {
        const unsigned short ELEMENT_NODE = 1;
        attribute Node parentNode;
        [TreatReturnedNullStringAs=Null] attribute DOMString nodeName;
        [Custom] Node appendChild([CustomReturn] Node newChild);
        void addEventListener(DOMString type, EventListener listener, optional boolean useCapture);
    };
}

{% endhighlight %}

# Proxy模式

通过Web IDL生成绑定代码通过proxy设计模式来完成对原始DOM对象的包装，从而使JavaScriptCore只能够访问到我们希望暴露的接口。Proxy模式的设计原理可以通过下图来阐释。

![proxy design pattern](/assets/images/posts/Proxy_pattern_diagram.png)

实例源代码如下，可以看到ProxyImage通过实现Image接口，包装了一个RealImage，当请求进入ProxyImage后会被转接到RealImage相应的方法。

{% highlight java %}
interface Image {
    public void displayImage();
}
 
//on System A 
class RealImage implements Image {
 
    private String filename = null;

    public RealImage(final String filename) { 
        this.filename = filename;
        loadImageFromDisk();
    }
 
    private void loadImageFromDisk() {
        System.out.println("Loading   " + filename);
    }

    public void displayImage() { 
        System.out.println("Displaying " + filename); 
    }
 
}
 
//on System B 
class ProxyImage implements Image {
 
    private RealImage image = null;
    private String filename = null;

    public ProxyImage(final String filename) { 
        this.filename = filename; 
    }

    public void displayImage() {
        if (image == null) {
           image = new RealImage(filename);
        } 
        image.displayImage();
    } 
}
 
class ProxyExample {
 
   public static void main(String[] args) {
        final Image IMAGE1 = new ProxyImage("HiRes_10MB_Photo1");
        final Image IMAGE2 = new ProxyImage("HiRes_10MB_Photo2");
 
        IMAGE1.displayImage(); // loading necessary
        IMAGE1.displayImage(); // loading unnecessary
        IMAGE2.displayImage(); // loading necessary
        IMAGE2.displayImage(); // loading unnecessary
        IMAGE1.displayImage(); // loading unnecessary
    }
 
}
{% endhighlight %}

# WebCore实现

对于WebCore来说也是一样，例如真正的WebCore对象类型是HTMLElement.h，生成的对象类型为JSHTMLElement (.h/.cpp)。在JSHTMLElement中包含了一个HTMLElement的对象，从而使绑定代码可以将方法调用代理给真正的DOM元素。

{% highlight cpp %}
void setJSHTMLElementInnerText(ExecState* exec, JSObject* baseObject, EncodedJSValue thisValue, EncodedJSValue encodedValue)
{
    JSValue value = JSValue::decode(encodedValue);
    UNUSED_PARAM(baseObject);
    JSHTMLElement* castedThis = jsDynamicCast<JSHTMLElement*>(JSValue::decode(thisValue));
    if (UNLIKELY(!castedThis)) {
        if (jsDynamicCast<JSHTMLElementPrototype*>(JSValue::decode(thisValue)))
            reportDeprecatedSetterError(*exec, "HTMLElement", "innerText");
        else
            throwSetterTypeError(*exec, "HTMLElement", "innerText");
        return;
    }
    // Get real HTMLElement
    HTMLElement& impl = castedThis->impl();
    ExceptionCode ec = 0;
    const String& nativeValue(valueToStringWithNullCheck(exec, value));
    if (UNLIKELY(exec->hadException()))
        return;
    // Call real setInnerText method
    impl.setInnerText(nativeValue, ec);
    setDOMException(exec, ec);
}
{% endhighlight %}

下面是通过脚本为元素的innerHTML赋值的具体调用栈，

{% highlight text%}
WebKit.dll!WebCore::HTMLElement::setInnerHTML 
WebKit.dll!WebCore::setJSHTMLElementInnerHTML <==生成的绑定代码 
JavaScriptCore.dll!JSC::callCustomSetter 
JavaScriptCore.dll!JSC::JSObject::put 
JavaScriptCore.dll!llint_slow_path_put_by_id 
JavaScriptCore.dll!llint_entry 
JavaScriptCore.dll!llint_entry 
JavaScriptCore.dll!callToJavaScript 
JavaScriptCore.dll!JSC::JITCode::execute 
JavaScriptCore.dll!JSC::Interpreter::executeCall 
JavaScriptCore.dll!JSC::call 
WebKit.dll!WebCore::JSMainThreadExecState::call
{% endhighlight %}

