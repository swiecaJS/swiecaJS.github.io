---
layout: post
title: "How to create content that fades in while scrolling"
date: 2018-12-13 20:11:22 +0200
categories: css animation javascript IntersectionObserver
---

This is a feature which is very popular among websites.

So quick **user story** what we want to achieve:
* When users scrolls down a page he should see an animated list of features
* When he scrolls back and forth the features are not disappearing. Animation should happen only once

A look at our website before finishing this task.

<p data-height="398" data-theme-id="0" data-slug-hash="qLbmEr" data-default-tab="result" data-user="khazarr" data-pen-title="Post about IntersectionObserver#1" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/qLbmEr/">Post about IntersectionObserver#1</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

We'll reuse animation from the last post
```css
    .pop-in {
      animation: pop 1.5s ease;
      animation-delay: .3s;
      animation-fill-mode: forwards;
    }

    /* animation */
    @keyframes pop {
    0% {
      transform: scale(0) translateY(100px);
      opacity: 0;
    }

    50% {
      transform: scale(1.05) translateY(-15px);
      opacity: 1;
    }

    100% {
      transform: scale(1) translateY(0px);
      opacity: 1;
    }
  }
```

So we need to add class ```.pop-in``` when an element is visible - but how to tackle this?


### IntersectionObserver to the rescue!

It's a powerful browser API that allows you to detect if an element is visible at the moment and react to change of it.

```javascript
const options = {
  root: document.querySelector('#scrollArea'),
  rootMargin: '0',
  threshold: 1.0
}
const observer = new IntersectionObserver(callback, options);

const callback = (entries, observer) => {}

// to start observing
observer.observe(DOMElement)
```

The options object (which is not required) can specify what is the root and define threshold od visibility. A threshold of 1.0 means that callback will be called when 100% of the element is visible in a **root** area.

However, in our example, we won't pass any options.

We start by creating our observer, we'll add a class when the element is visible. in ```IntersectionObserverEntry``` we have couple values. In our case we need just boolean **isIntersecting**.
```javascript
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('pop-in');
      }
    })
  })
```
Now we just need to query all ```features``` and ```.observe()``` them.

```javascript
  const features = document.querySelectorAll('.feature')
  features.forEach(feature => {
    observer.observe(feature)
  })
```

Working example below:


<p data-height="392" data-theme-id="0" data-slug-hash="aPdWrq" data-default-tab="result" data-user="khazarr" data-pen-title="Post about IntersectionObserver#2" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/aPdWrq/">Post about IntersectionObserver#2</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

---
*resources*
* [Tutorial about IntersectionObserver](https://alligator.io/js/intersection-observer/)
* [MDN Docs - IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
* [MDN Docs - IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry)







