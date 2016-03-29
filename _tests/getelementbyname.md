---
layout: test
title: DOM getElementByName
---

<div class="section">
    <div id="test">
        <p name="test">This is a paragraph with name="test"</p>
        <custom name="test">This is a custom tag with name="test"</custom>
        <p><input name="test">This is an input with name="test"</p>
        <p><img name="test">This is an image with name="test"</p>
    </div>


    <h3>Source Code</h3>
    <pre class="brush: js">
        &lt;div id=&quot;test&quot;&gt;
        &lt;p name=&quot;test&quot;&gt;This is a paragraph with name=&quot;test&quot;&lt;/p&gt;
        &lt;custom name=&quot;test&quot;&gt;This is a custom tag with name=&quot;test&quot;&lt;/custom&gt;
        &lt;p&gt;&lt;input name=&quot;test&quot;&gt;This is an input with name=&quot;test&quot;&lt;/p&gt;
        &lt;p&gt;&lt;img name=&quot;test&quot;&gt;This is an image with name=&quot;test&quot;&lt;/p&gt;
        &lt;/div&gt;

        var x = document.getElementsByName('test');
        var output = [];
        for (var i = 0; i < x.length; i += 1) {
                output.push(x[i].nodeName);
        }
        document.getElementById('output').innerHTML = output.join(',');
</pre>
<input type="button" value="Execute Test" onclick="test()" />
    <h3>Output</h3>
    <p id="output"></p>
    <script>
        function test() {
            var x = document.getElementsByName('test');
            var output = [];
            for (var i = 0; i < x.length; i += 1) {
                output.push(x[i].nodeName);
            }
            document.getElementById('output').innerHTML = output.join(',');
        }
    </script>

</div>