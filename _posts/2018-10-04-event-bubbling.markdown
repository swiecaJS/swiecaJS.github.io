---
layout: post
title:  "DOM Events: Bubbling and Capturing"
date:   2018-10-04 20:18:22 +0200
categories: javascript DOM-events
---
Bubbling means that:

* When an event happens on an element, it first runs the handler directly on it. Then it is moved to the parent, and it goes up to the end of the ancestors, just like bubbles in water.
  
```html
<form>
  <input id="name" type="text">
  <input id="surname" type="text">
  <div>
    <p id="clicker" >Click me please!</p>
  </div>
</form>

```
Thanks to that we can for example have an event listener on a ```<form>``` and listen on all child events in one global handler.

### How to stop it?

The solution for it is ```event.stopPropagation()```. It will stop event propagation but **only** on the current handler. That means when you have multiple handlers for an event, you will stop only current one

* Use it when absolutly necesseary - it may be really hard for debuging/refactoring in future. 
* When you think you may need it - go for *custom events*!

### Capturing - the inverse of bubbling

```addEventListener(..., true)``` can take up third parameter - which inverse the way of event journey. I.e. the event will propagate down in ancestors hierarchy.


### Examples

* Bubbling and capturing - see it in action. Click elements to see how event propagates
<br>
<p data-height="415" data-theme-id="0" data-slug-hash="LgEMQX" data-default-tab="result" data-user="khazarr" data-pen-title="#Bubbling" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/LgEMQX/">#Bubbling</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

<br>
* Propagation stopped - in this example the listener on ```level-four``` has an ```event.stopPropagation``` inside. See what it changes
<br>
<p data-height="420" data-theme-id="0" data-slug-hash="dgPaJY" data-default-tab="result" data-user="khazarr" data-pen-title="#Bubbling - stop propagation" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/dgPaJY/">#Bubbling - stop propagation</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>