---
layout: test
title: DOM element children
---

<div class="section">
    <script>
        function test() {
            alert('testRoot children length is: ' + document.getElementById('testRoot').children.length);
        }
    </script>
    <div id="testRoot">
        <em>
            This DIV with id="testRoot" starts with an em
        </em> and HAS EMPTY TEXT NODES.
        <p id="testPar">
            and then comes a paragraph with id="testPar"
            <strong>which has a strong tag</strong>
            nested in it.
        </p>
        <!-- This is a comment -->
        <div>
            And after a comment comes this div.
            <p>And in this nested paragraph I remind you that there are empty text nodes between the elements.</p>
        </div>
    </div>

    <h3>Source Code</h3>
    <pre class="brush: js">
        alert('testRoot children length is: ' + document.getElementById('testRoot').children.length);
    </pre>

    <input type="button" id="test" value="Execute Test" onclick="test()" />

</div>