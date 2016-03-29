---
layout: test
title: leak test 1
---

<div class="section">
    <script>
    /*global setTimeout */
    (function(limit, delay) {
        var queue = new Array(10);
        var n = 0;

        function makeSpan(n) {
            var s = document.createElement('span');
            document.body.appendChild(s);
            var t = document.createTextNode(' ' + n);
            s.appendChild(t);
            return s;
        }

        function process(n) {
            queue.push(makeSpan(n));
            var s = queue.shift();
            if (s) {
                s.parentNode.removeChild(s);
            }
        }

        function loop() {
            if (n < limit) {
                process(n);
                n += 1;
                setTimeout(loop, delay);
            }
        }

        loop();
    })(10000, 10);
    </script>
    <h3>Source Code</h3>
    <pre class="brush: js">
        /*global setTimeout */
            (function (limit, delay) {
                var queue = new Array(10);
                var n = 0;

                function makeSpan(n) {
                    var s = document.createElement('span');
                    document.body.appendChild(s);
                    var t = document.createTextNode(' ' + n);
                    s.appendChild(t);
                    return s;
                }

                function process(n) {
                    queue.push(makeSpan(n));
                    var s = queue.shift();
                    if (s) {
                        s.parentNode.removeChild(s);
                    }
                }

                function loop() {
                    if (n < limit) {
                        process(n);
                        n += 1;
                        setTimeout(loop, delay);
                    }
                }

                loop();
            })(10000, 10);
</pre>

</div>
