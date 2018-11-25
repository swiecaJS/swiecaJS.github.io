---
layout: post
title: "Observable objects pt. II (Reactive DOM)"
date: 2018-11-25 11:39:22 +0200
categories: javascript
---


In the previous post, we learned how we can define and react to change in object properties. Let's use that knowledge to create reactive form helper. Let's reuse ```observe()``` function.

```javascript

function observe (obj, key, callback) {
  let observedValue = obj[key]

  Object.defineProperty(obj, key, {
    get () {
      return observedValue
    },
    set (newVal) {
      callback(newVal)
      observedValue = newVal
      return observedValue
    }
  })
}

```

### User Story
Our task will be to bind ```<input>``` and some text tag, like ```<h3>```. We would like to change text value when input value changes. Also, it will be nice to set some starting values for binding

```html
<h3 swiecajs-text="name">Name</h3>
<input swiecajs-model="name" placeholder="name" type="text">
```
Our DOM pair will look like this. We will use our custom ```swiecajs-...``` property for binding. Let's see how we can create a configuration object

```javascript
const obj = {
    selector: 'name',
    data: 'John',
    placeholder: 'enter name'
  }
```

Right, so we need to select those values and assign the initial state. Let's also keep initial data for the input because we would like to display it when an input is empty.
```javascript

    const initialData = obj.data

    const input = document.querySelector(`[swiecajs-model="${obj.selector}"]`)
    input.placeholder = obj.placeholder

    const output = document.querySelector(`[swiecajs-text="${obj.selector}"]`)
    output.innerHTML = obj.data

```
Now we can use our ```observe()``` function to track changes in the object. We will also use ```addEventListener``` to track changes of input and assign it to the ```obj.data``` property.

When obj.data changes value, DOM will be updated

```javascript
// observe changes on input
input.addEventListener('input', (ev) => {
  obj.data = ev.target.value ? ev.target.value : initialData
})

// react to changes
observe(obj, 'data', (newValue) => {
  output.innerHTML = newValue
})
```

And that's it! We can put everything in action and create ```new``` instances of a reactive pair. 

```javascript
 const NameInput = new swiecaJsInputs({
    selector: 'name',
    data: 'John',
    placeholder: 'enter name'
  })
```

Let's see a working example below


<p data-height="476" data-theme-id="0" data-slug-hash="eQrMGy" data-default-tab="result" data-user="khazarr" data-pen-title="reactive inputs" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/eQrMGy/">reactive inputs</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


---
*resources*
* [Damian Dulisz post about Observable objects](https://www.monterail.com/blog/2016/how-to-build-a-reactive-engine-in-javascript-part-1-observable-objects)
* [Build your own Vue.js github repo](https://github.com/jsrebuild/build-your-own-vuejs/blob/master/book/chapter1.md)
* [MDN Docs about Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

