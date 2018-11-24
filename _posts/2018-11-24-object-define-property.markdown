---
layout: post
title: "Observable objects"
date: 2018-11-24 11:39:22 +0200
categories: javascript
---

To understand what's happening under the hood of modern JS frameworks (like Vue.js) let's focus on how to make an object property observable.

Let's define our task. We have this object
```javascript
const dog = {
 name: 'Sparky',
 isSleeping: false
}
```

We would like to be notified, whenever our dog is hungry. How to do that?

**Object.defineProperty() to the rescue!**

We can make a wrapper for this object, keep value (thanks to closures) and notify everytime something happens.

```javascript
function Dog (name) {
  let isSleeping = false
  Object.defineProperty(this, 'isSleeping', {
    get () {
      console.log(`checking if ${name} is sleeping`)
      return isSleeping
    },
    set (newVal) {
      console.log(`${name} is${newVal ? '' : ' not'} sleeping`)
      isSleeping = newVal
    }
  })
}

const dog = new Dog('Sparky')

console.log(dog.isSleeping) 
dog.isSleeping = true
console.log(dog.isSleeping)
dog.isSleeping = false
```

Thanks to access to get/set hooks, we can do everything we want to track changes. But this wrapper requires to use ```new``` keyword. But could we tackle this problem without changing the initial API of dog?

 We would like to track the change of ```isHungry``` property. Let's try to write a function that will take an object and key as a parameter

```javascript
const dog = {
  name: 'Sparky',
  isSleeping: false
}

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

observe(dog, 'isSleeping', (key) => {
  console.log(`isSleeping value changed to ${key}`)
})

dog.isSleeping = true
dog.isSleeping = false
```

Our function's callback will be triggered everytime the value of property changes, and we do not have to change initial API!


---
*resources*
* [MDN Docs about Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

