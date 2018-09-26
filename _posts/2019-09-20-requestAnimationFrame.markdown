---
layout: post
title:  "request Animation Frame() - another look at animation"
date:   2018-09-20 19:39:22 +0200
categories: javascript animation time setInterval
---


The oldschool & Classic way of animating content in using ```SetTimeout()``` or ```setInterval()``` .
But those function can heavily decrease performance. Browser wants to repaint screen in approximately 60FPS (or 16.6ms). But it's not possible when the stack is not empty. Every call to mentioned functions put a task onto a queue, which is moved to a stack after a specified time. So it might happen that if you abuse them **You may block UI**. Play with it by adding junk to stack - in this case, a lot of factorial recursive calls.

<p data-height="296" data-theme-id="0" data-slug-hash="gdqdxy" data-default-tab="result" data-user="khazarr" data-pen-title="setInterval()" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/gdqdxy/">setInterval()</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


Another example uses ```requestAnimationFrame ``` for drawing. It's a simple solution with recursive calls

Some theory at the beginning

``` requestAnimationFrame()```

- **parameter**
  - a callback function that will be called at the next repaint
  - callback first parameter has a value of ```performane.now()```
  - i.e ```requestAnimationFrame(console.log)``` will print value of ```performance.now()```
- **return**
  - an integer id value which you can use to ```cancelAnimationFrame()```

Ok, let's see it in action, the same scenario. Play with it and compare your performance when the stack is heavily loaded with factorial recursive calls

<p data-height="265" data-theme-id="0" data-slug-hash="QVYVpw" data-default-tab="result" data-user="khazarr" data-pen-title="requestAnimationFrame - drawing" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/QVYVpw/">requestAnimationFrame - drawing</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

*PS. It might happen that your machine is cutting edge piece fo tech, and you'll need to add some more recursive calls to see some UI blocking*

*PS2. actually, browsers may throw ```Uncaught RangeError: Maximum call stack size exceeded``` so you won't see this effect of UI blocking, so just enjoy animation ;)*




