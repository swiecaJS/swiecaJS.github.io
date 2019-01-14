---
layout: post
title: "Understaning Vue custom directives"
date: 2019-01-14 15:38:22 +0200
categories: javascript vue
---

The directives are commands attached to a DOM element, which allows us to run some logic. If you are using Vue, you must have used ```v-if``` or ```v-show``` for conditional rendering or ```v-for``` for list rendering. 
As we can see they are prefixed with ```v-``` prefix.

```html
    <p v-show="clicked">Now you see me</p>
    <button v-on:click="clicked = !clicked">click me</button>
```

In the above code we will show text depending on the value of ```clicked``` variable. Let's write our custom directive to better understand this topic.

### The basics

You can create custom directive globally using
```javascript
Vue.directive(directiveName, configObject or callbackFunction)

// creates global directive
Vue.directive('example', (el, binding, vNode, oldVnode) => {
    // your logic
})
```
Or directly in component
```javascript
directives: {
  example (el, binding, vNode, oldVnode) {
      // your logic
    }
  }
}
```
We start with passing an funtion as a second parameter and foucs on directive arguments.

* el - The DOM element directive is bound.
* binding - an object with following properties. We'll focus on the most important to understand basics
  * name - name of directive without ```v-``` prefix
  * value - value passed to directive, for example ```v-example='2+2'```, the value will be equal to ```4```
  * expression - value passed, but as a string, for example ```v-example='2+2'```, the value will be equal to ```'2+2'```
* vnode - virtual node produced by Vue compiler
* oldVnode - previous virtual node, only availbile in ```update``` and ```componentUpdated``` **hooks** (we'll get to them later).


### Build your own v-show!

Ok so let's use that knowledge to build our first custom directive. We'll create our own version of conditional rendering.

Let's remind us our HTML setup
```html
    <div id="container">
      <h1>Vue Directives</h1>
      <p v-show-2="clicked">Now you see me</p>
      <button @click="clicked = !clicked">click me</button>
    </div>
```

Ok we are passing ```clicked``` value to our directive, and want to alter ```display``` property

```javascript
    Vue.directive('show-2', (el, binding) => {
      el.style.display = binding.value ? 'inherit' : 'none'
    })
```

As we can see - if the value of binding - in our example ```clicked``` variable will be ```false```, display style of the element will be equal to ```none```.

<p data-height="253" data-theme-id="0" data-slug-hash="WLzzQy" data-default-tab="result" data-user="khazarr" data-pen-title="Vue directive post" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/WLzzQy/">Vue directive post</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### Reuse intersection observer

Now we'll write a directive that will add class to an element when it will be visible. Let's reuse code from [Intersection Observer post]({% post_url 2018-12-13-intersection-observer %}).

For sake of simplicity, let's create a new intersection instance for every occurrence. We will pass a class name that directive will add to an element.

```javascript
  Vue.directive('intersect', (el, binding) => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add(binding.value);
        }
      })
    })
    observer.observe(el)
  })
```

```html
<div class="feature" v-intersect="'pop-in'">
  <p>COOL</p>
  <i class="fab fa-angellist"></i>
</div>
```

Ok, it will work, but let's rewrite it that the config object will be passed to the directive, observer and custom bg color, because we can pass also **objects** to directives:

```javascript
    new Vue({
      el: '.wrapper1',
      data () {
        return {
          observer: null,
          classToAdd: 'pop-in'
        }
      },
      methods: {
        createObserverConfig (bgColor) {
          return {
            observer: this.observer,
            bgColor
          }
        }
      },
      created () {
        this.observer = new IntersectionObserver((entries) => {
          entries.forEach(entry => {
            if (entry.isIntersecting) {
              entry.target.classList.add(this.classToAdd);
            }
          })
        })
      },
      directives: {
        intersect (el, binding) {
          el.style.backgroundColor = binding.value.bgColor
          binding.value.observer.observe(el)
        }
      }
    })
```
```html
  <div class="feature" v-intersect="createObserverConfig('red')">
    <p>COOL</p>
    <i class="fab fa-angellist"></i>
  </div>
```

Let's see them in action

<p data-height="265" data-theme-id="0" data-slug-hash="LMddyR" data-default-tab="result" data-user="khazarr" data-pen-title="Vue directive post 2" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/LMddyR/">Vue directive post 2</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


### The hooks

We have five hooks that we can use when describing directive:
* bind - called only once, when directive bound to element
* inserted - when an element is inserted to the parent element (that does not mean document presence)
* update
* componentUpdated -when component's vNode or **any of its child** have been updated 
* unbind - called only once, when directive unbound

To observe in practice play with below example. Observe that because ```<input>``` uses ```v-show``` it's automatically bind when pages load. 
```html
    <article id="container">
      <button v-on:click="btnClicked = !btnClicked">Add element</button>
      <input placeholder="write something" type="text" v-model="inputText" v-show="btnClicked" v-custom>
      <p  v-custom v-if="btnClicked" >input text: {{inputText}}</p>
    </article>
```
```javascript
    new Vue({
      el: '.wrapper',
      data () {
        return {
          inputText: null,
          btnClicked: false
        }
      },
      directives: {
        custom: {
          bind: function (el, binding, vNode) {
            M.toast({ html: `${vNode.tag} bind` })
          },
          inserted: function (el, binding, vNode) {
            M.toast({ html: `${vNode.tag} inserted` })
          },
          update: function (el, binding, vNode) {
            M.toast({ html: `${vNode.tag} update` })
          },
          componentUpdated: function (el, binding, vNode) {
            M.toast({ html: `${vNode.tag} componentUpdated` })
          },
          unbind: function (el, binding, vNode) {
            M.toast({ html: `${vNode.tag} unbind` })
          }
        }
      }
    })
```

<p data-height="415" data-theme-id="0" data-slug-hash="REMMBx" data-default-tab="result" data-user="khazarr" data-pen-title="Vue directive post 4" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/REMMBx/">Vue directive post 4</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
---
*resources*
* [Vue.js docs](https://vuejs.org/v2/guide/custom-directive.html)












