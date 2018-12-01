---
layout: post
title: "console.log custom styling"
date: 2018-12-01 17:39:22 +0200
categories: javascript
---

Have you ever wondered how Facebook displays this cool custom message in the browser's console? Well, I did.

![Facebook selfXSS alert](/assets/img/console/fb.PNG)

It happens that good old ```console.log``` can take a second parameter and use some string interpolation.

The basic one is with the ```%s``` for strings. For example

```javascript
const name = 'John'
console.log('hello %s' , name) // prints hello John
```

But what is really cool, you can use ```%c``` for custom CSS in the ```console.log``` output. For example

```javascript
const msg = 'Hello'
const css = 'font-size: 50px;color: #EDFFD9;text-shadow: 2px 2px black;;background: linear-gradient(to right, #DB9D47, #FF784F);padding:20px'
console.log('%c%s' , css, msg)
```
![Example of custom console styling](/assets/img/console/1.PNG)

But I think this might be cleaned up a bit

```javascript
const msg = 'Hello'
const css = [
  'font-size: 50px',
  'color: #EDFFD9',
  'text-shadow: 2px 2px black',
  'background: linear-gradient(to right, #DB9D47, #FF784F)'
  'padding:20px'
].join(';')
console.log('%c%s' , css, msg)
```

So that's how it's made ;)

---
*resources*
* [MDN Docs](https://developer.mozilla.org/pl/docs/Web/API/Console)
* [Medium post about this topic](https://itnext.io/console-rules-b30560fc2367)
* [SVG animations in browsers console](https://hackernoon.com/i-made-a-discovery-svg-and-svg-animations-in-the-js-console-are-doable-6c367c95726a)

