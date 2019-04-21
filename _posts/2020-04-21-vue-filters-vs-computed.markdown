---
layout: post
title: "Are Vue.js filters really that bad?"
date: 2019-04-21 14:38:22 +0200
categories: javascript vue
---


Recently during the code review, I was debating with my colleagues about extracting some functions as vue filters. Those functions were strictly related to text formatting. In addition, they were used in multiple components. For me, it seemed natural to move those functions to global `Vue.filter` and leverage text formatting in a template. 

The problem is that this move might come with a **price**. Let's dive in and check how much it might cost!

### What are Vue filters?

First, a quick reminder of how to work with filters.
You can create them in a component:
```javascript
filters: {
  firstLetter (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0)
  }
}
```
Or globally before creating the `Vue` instance:
```javascript
Vue.filter('firstLetter', (value) => {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0)
})

new Vue({
  //...
})
```

Later on, you can use them using the **pipe** symbol -> `|` 

{% highlight html %}
{% raw %}
<p>{{userName | firstLetter}}</p>
<your-component :some-props="userName | firstLetter"/>
{% endraw %}
{% endhighlight %}

What is **very important** to remember: You cannot access the current component instance by `this`!
In filters `this` is bound to the global object (window), not to the component instance. This is not an accident because filters **must not be used to generate side effects**. Their only purpose is to perform text formatting.

### What's wrong with them?

Ok, so the first problem with filters is they lack any caching mechanism. It means that they work just like `methods` in Vue. They need to reevaluate for **every call**, even when they have the same input. 

Let's look at this scenario:

{% highlight html %}
{% raw %}
<template>
  <section>
    <p>{{ user.name | capitalize }} {{ user.surname | capitalize }}</p>
    <p>{{ user.name | capitalize }} {{ user.surname | capitalize }}</p>
    <p>{{ user.name | capitalize }} {{ user.surname | capitalize }}</p>
    <!-- 3 reevaluations! -->

    <p>{{ userDataCapitalized }}</p>
    <!-- The value is stored in cache, changes only when data change -->
    <p>{{ userDataCapitalized }}</p>
    <p>{{ userDataCapitalized }}</p>

  </section>
</template>
<script>
  const _capitalize = value => {
    if (!value) return "";
    value = value.toString();
    return value.charAt(0).toUpperCase() + value.slice(1);
  };
  export default {
    data() {
      return {
        user: {
          name: "john",
          surname: "doe"
        }
      };
    },
    filters: {
      capitalize(value) {
        console.log("filter called");
        return _capitalize(value);
      }
    },
    computed: {
      userDataCapitalized() {
        console.log("computed called");
        return `${_capitalize(this.user.name)} ${_capitalize(
          this.user.surname
        )}`;
      }
    }
  }
</script>
{% endraw %}
{% endhighlight %}

In console, you'll see only one `console.log` from the computed property, and **six** from filter! That was expected. In such a case, it's much more efficient to create computed property in the component.

### But what happens when you need to pass some parameters? 

In this scenario, we would like to trim and capitalize text. But we would like to have the ability to change length.

```javascript
const _trimAndCapitalize = (text, length ) => {
  console.log('doin work!')
  if (!text) return "";
  text = text.toString();
  return text.charAt(0).toUpperCase() + text.slice(1,length);
}
```

Let's imagine we have a lot of different edge cases in our component and creating a computed property for each case is not an option. So the question is:

#### Which solutiom will be the most performant?

Calling filter? Calling method? Assigning function reference to the computed property? Let's find out!

```javascript
export default {
  data() {
    return {
      text1: "ullam velit quis eum aliquam illum numquam ipsa.",
      text2: "amet consectetur, adipisicing elit.",
      text3: "res sit, cupiditate sapiente",
      text4: "adipisicing elit. Maiores sit, illum numquam ipsa."
    };
  },
  filters: {
    trimAndCapitalizeFilter(text, length) {
      console.log("filter called");
      return _trimAndCapitalize(text, length);
    }
  },
  methods: {
    trimAndCapitalizeMethod(text, length) {
      console.log("method called");
      return _trimAndCapitalize(text, length);
    }
  },
  computed: {
    trimAndCapitalizeComputed() {
      console.log("computed called");
      return _trimAndCapitalize;
    }
  }
}
```

This is our setup. In the template, we have the following code:
{% highlight html %}
{% raw %}
    <section>
      <h3>Filters</h3>
      <p>{{ text1 | trimAndCapitalizeFilter(2)}}</p>
      <p>{{ text2 | trimAndCapitalizeFilter(11)}}</p>
      <p>{{ text3 | trimAndCapitalizeFilter(3)}}</p>
      <p>{{ text4 | trimAndCapitalizeFilter(5)}}</p>
      <!-- filter called 4 times | method doing work 4 times -->
      <hr />
      <h3>Computed</h3>
      <p>{{ trimAndCapitalizeComputed(text1, 2) }}</p>
      <p>{{ trimAndCapitalizeComputed(text2, 11) }}</p>
      <p>{{ trimAndCapitalizeComputed(text3, 3) }}</p>
      <p>{{ trimAndCapitalizeComputed(text4, 5) }}</p>
      <!-- computed called 1 times | method doing work 4 times -->
      <hr />
      <h3>Methods</h3>
      <p>{{ trimAndCapitalizeMethod(text1, 2) }}</p>
      <p>{{ trimAndCapitalizeMethod(text2, 11) }}</p>
      <p>{{ trimAndCapitalizeMethod(text3, 3) }}</p>
      <p>{{ trimAndCapitalizeMethod(text4, 5) }}</p>
      <!-- component method called 4 times | actual method doing work 4 times -->
    </section>
{% endraw %}
{% endhighlight %}

As expected, each time text formatting was needed, the `_trimAndCapitalize()` function was invoked. But what is interesting, the computed property was called only **once** in this flow. 

<hr>
### Performance tests

Before we dig deeper, let's do some jsPerf tests, to determine which setup is the most efficient. [You can it try yourself here!](https://jsperf.com/vue-js-method-vs-filters-vs-computed)
![js perf results](/assets/img/vue-filters-vs-computed/jsperf.png)

What was surprising to me, storing a reference to a function in the computed property was the slowest one. This suggests that this caching mechanism might have more significant overhead than filters. But I think the most important fact is that the results **are really close to each other**. Given this data, we can assume that arguing which solution is better is actually *bikeshedding* (it doesn't really matter xD). 

### Time to look under the hood!
Anyway, this made me curious. Let's start our investigation by translating our template to `render()` function.

At the begging, we will rewrite these cases:
```javascript

// method
render(createElement) {
  return createElement('p', this.trimAndCapitalizeMethod(this.text1, 1))
}
// computed
render(createElement) {
  return createElement('p', this.trimAndCapitalizeComputed(this.text1, 1))
}
```
With `method` and `computed` property, this translation is rather straightforward. The tricky question is how to access **filters** using `this`? Watch out, the following code doesn't work!

```javascript
render(createElement) {
  return createElement('p', this.trimAndCapitalizeFilter(this.text1, 1))
}
```
![render error message](/assets/img/vue-filters-vs-computed/render_error.png)

`this.trimAndCapitalizeFilter` has value of `undefined`. Do you have an idea why? If not, no worries. We can use helper function `Vue.compile()`. It will output the translation of a template as a `render()` function call.

{% highlight javascript %}
{% raw %}

const filter = Vue.compile("<p>{{ text1 | trimAndCapitalizeFilter(2)}}</p>");
console.log(filter.render);

/* console output
ƒ anonymous() {
with(this){return _c('p',[_v(_s(_f("trimAndCapitalizeFilter")(text1,2)))])}
}
*/
{% endraw %}
{% endhighlight %}

Yeah, we have something! But how to decode what those `_` prefixed functions are? We won't find the answer in docs (belive me, I tried xD). If you look through vue js source code you'll find this:

```javascript
  function installRenderHelpers (target) {
    target._o = markOnce;
    target._n = toNumber;
    target._s = toString;
    target._l = renderList;
    target._t = renderSlot;
    target._q = looseEqual;
    target._i = looseIndexOf;
    target._m = renderStatic;
    target._f = resolveFilter;
    target._k = checkKeyCodes;
    target._b = bindObjectProps;
    target._v = createTextVNode;
    target._e = createEmptyVNode;
    target._u = resolveScopedSlots;
    target._g = bindObjectListeners;
    target._d = bindDynamicKeys;
    target._p = prependModifier;
  }
```

Knowing this, we can rewrite it to something understandable:
```javascript
createElement('p', [createTextVNode(toString(resolveFilter("trimAndCapitalizeFilter")(text1,2)))])
```

Perfect! Now we know that in `render()` method we can access filters using `this._f(filterName)`. We can write our working render function like this:
```javascript
render(createElement) {
  createElement('p', this._f('trimAndCapitalizeFilter')(this.text1, 2))
}
```

But can we do better? Let's check in code what `resolveFilter` actually stands for!

```javascript
  function resolveFilter (id) {
    return resolveAsset(this.$options, 'filters', id, true) || identity
  }
```

Bingo! We see that our filter is stored in `$options`! So our final `render()` function will look like this:
```javascript
render(createElement) {
  return createElement('p', this.$options.filters.trimAndCapitalizeFilter(this.text1, 2))
}
```

### Final thoughts

* Vue `filters` are just methods that are stored in a different part of the Vue instance (`$options`). 
* They are not bound to the current instance. Therefore, you cannot use `this` to reference local properties. 
* If you cannot create computed property for each variable because you need to pass additional parameters - it doesn't really matter what you choose (Filters, methods, storing a reference to external function in computed property). It shouldn't affect the code performance. I would personally go for `filters` because I really like that syntax ;) 
* Have in mind that you should always aim for `computed` properties if possible.
<br>
<br>
P.S What's exactly happening underneath `computed` properties needs and will be investigated later. 

---

