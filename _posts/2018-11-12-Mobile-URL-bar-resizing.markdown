---
layout: post
title: "Mobile - resizing height beacuse of dissapearing scroll bar"
date: 2018-11-12 21:49:22 +0200
categories: rwd css
---

Recently I have a task to prepare a modal that will be displayed on top of a page. Nothing really fancy.
So by default I will use below css to position a modal:
```css
  .modal {
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
  }
```
If you check on desktop everything looks good


But there is major problem on mobile, due to hiding URL search bar in chrome.

![Mobile screenshot with a problem](/assets/img/mobile-url/bug.png)

The solution for that is to **use 100vh**. It's actually recommended by google.
```css
  .modal {
    position: fixed;
    left: 0;
    top: 0;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
  }
```

After that it looks much better.


![Mobile screenshot with a problem](/assets/img/mobile-url/good.png)


### Codepen examples
> Fixed one
<p data-height="265" data-theme-id="0" data-slug-hash="EOvQea" data-default-tab="result" data-user="khazarr" data-pen-title="modal 100vh height" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/EOvQea/">modal 100vh height</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

> Buggy on mobile one
<p data-height="265" data-theme-id="0" data-slug-hash="GwvQxE" data-default-tab="result" data-user="khazarr" data-pen-title="modal 100% height" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/GwvQxE/">modal 100% height</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
---
*resources*
* [Google post about url - resizing](https://developers.google.com/web/updates/2016/12/url-bar-resizing)
* [Example of url bar resiza](https://bokand.github.io/demo/urlbarsize.html)
