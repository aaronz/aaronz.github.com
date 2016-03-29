---
layout: test
title: Hit Testing
---

<div class="section">

    <input id="button" type="button" value="click" onclick="alert('clicked')" />
    <div id="transparent-div"></div>
    <h3>Source Code</h3>
    <pre class="brush: js">
        #transparent-div{
            width:400px;
            height:400px;
            border-bottom-color:red;
            border-bottom-width:1px;
            border-style:solid;
            z-index:0;
        }

        #button{
            position:absolute;
            top:200px;
            left:50px;
            z-index:-1;
        }
    </pre>

</div>