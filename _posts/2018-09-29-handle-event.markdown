---
layout: post
title:  "Another look at addEventListener"
date:   2018-09-29 17:39:22 +0200
categories: javascript DOM-events
---

Everybody knows this *classic* way of handling events. We use it everyday

```javascript
element.addEventListener('click', handlerFunc)
```
Interesting fact is that actually we can pass as a second arugment an object. When event fires, the ```handleEvent``` method will be called

```javascript
<script>
element.addEventListener('click', {
  handleEvent(event) {
    // do stuff with event
  }
})
<script>
```
A simple example of usage, click the button to see an output.

<p data-height="161" data-theme-id="0" data-slug-hash="KGwyPE" data-default-tab="result" data-user="khazarr" data-pen-title="Events #1" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/KGwyPE/">Events #1</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


This approach gives us an oppurtunity to create an object (or even a ES6 class) which will store multiple handlers for events. I find it quite clear to use. Check out example

```javascript
  class BtnHandler {
    handleEvent(event) {
      switch(event.type) {
        case 'mousedown':
          // do stuff on mouse down
          break;
        case 'mouseup':
        // do another stuff on mouse up
          break;
        case 'mouseenter':
        // do another stuff on mouseenter
          break;
        case 'mouseleave':
        // do another stuff on mouseleave
          break;
      }
    }
  }

  let handler = new BtnHandler();
  el.addEventListener('mousedown', handler);
  el.addEventListener('mouseup', handler);
  el.addEventListener('mouseenter', handler);
  el.addEventListener('mouseleave', handler);
```

<p data-height="169" data-theme-id="0" data-slug-hash="mzyqOx" data-default-tab="result" data-user="khazarr" data-pen-title="Events #2" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/mzyqOx/">Events #2</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

That is another look at event handling.

---

### Summary - options for addEventListener
```javascript
// passing a reference to a function or an object which posses handleEvent method
el.addeventListner('eventName', functionReference)


// or creating an anonymous callack
el.addeventListner('eventName', event => {
  //do stuff with it
})
```