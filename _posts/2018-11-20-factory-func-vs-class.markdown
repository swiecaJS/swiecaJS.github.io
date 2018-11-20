---
layout: post
title: "Factory functions vs classes"
date: 2018-11-20 17:49:22 +0200
categories: javascript-patterns javascript
---

A short comparison between those two patterns in JS, and why factory functions might be easier to use.

### Classes

Imagine you have handles with a lot of popups on your page, and to have logic in one place you decide to create a class Modal.

```javascript
class Modal {
  constructor (modalSelector) {
    this.modal = document.querySelector(modalSelector)
    this.isVisible = false
  }
  show () {
    this.modal.style.display = 'flex'
    this.isVisible = true
  }
  hide () {
    this.modal.style.display = 'none'
    this.isVisible = false
  }
}

const popup = new Modal('#ads-popup')
```

Ok, everything looks fine. Let's open this popup when the user clicks on a button. What could be your first thought?

```javascript
const button = document.querySelector('#popup-btn')

button.addEventListener('click', popup.show)
```
ok, so you are passing a reference to a function which will be called on click. But the question for you. **Will it works?**

The answer is no.
> Uncaught TypeError: Cannot read property 'style' of undefined

The reason for that, we the *this* keyword in a callback function reflects the DOM element, not our class. We can fix it with ```.bind()```.

```javascript
const button = document.querySelector('#popup-btn')

button.addEventListener('click', popup.show.bind(popup))
```

Of course, we might use arrow functions and call popup directly, but let's look what else is available

### Factory functions

```javascript
const modalFactory = (modalSelector) => {
  const modal = document.querySelector(modalSelector)
  let isVisible = false
  return {
    show: () => {
      modal.style.display = 'flex'
      isVisible = true
    },
    hide: () => {
      modal.style.display = 'none'
      isVisible = false
    },
    isVisible: () => isVisible
  }
}
```

What's changed? We simply have a function that returns an object. We have access only to functions that are revealed. Thank's to closures we can change the 'private' variables - ```isVisible``` in this case. But we **cannot** access them directly. That is possible in the previous class example.

In addition, it will work with event listener without binding ```this```. That's cool!

```javascript
const popup = modalFactory('#popup')
const button = document.querySelector('#popup-btn')

button.addEventListener('click', popup.show)
```

### Summary

* Factory functions solves a problem with tricky words in JS world: ```this``` and ```new```.
* They might not be as efficient as classes performance wise. But unless you have millions of instances you shouldn't worry about it ;)

---
*resources*
* [Fun Fun Function about factory functions](https://www.youtube.com/watch?v=ImwrezYhw4w)

