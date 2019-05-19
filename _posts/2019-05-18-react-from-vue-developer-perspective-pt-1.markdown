---
layout: post
title: "Trying to use Vue.js patterns in React. How to compensate slots and named slots?"
date: 2019-05-18 14:38:22 +0200
categories: javascript vue react
---


For my commercial projects I mainly use Vue.js. But sometimes when it's needed I also develop web apps in React. I am no React expert at all, but I want to share my thoughts about similar patterns. Today I'll focus on how to compensate `<slot>` in React.

If you need to catch up with Vue slots first - [check out my other post about it]({% post_url 2019-02-15-vue-scoped-slots %})

### The simplest case

We want to compose our component like this

```javascript

const App = () => {
  return(
    <BaseLink to={"/home"}>
      Some Text
    </BaseLink>
  );
};

```

This is pretty simple in React, because by default everything enclosed by components tag is assigned to `props.children`. So in such case our `<BaseLink>` component might look like this:

```javascript
const BaseLink = props => {
  return(
   <a href={props.to} class="base-link">{props.children}</a>
  );
};
```

We can clean up this a bit, using ES6 destructuring

```javascript
const BaseLink = ({to, children}) => (
  <a href={props.to} class="base-link">{props.children}</a>
);
```

See example below:

<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="js,result" data-user="khazarr" data-slug-hash="RmgepR" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Basic Vue slot made in React">
  <span>See the Pen <a href="https://codepen.io/khazarr/pen/RmgepR/">
  Basic Vue slot made in React</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>


### Named slots

Ok now things are getting a little bit tricky. Let's start with defining what we would like to achieve. In Vue our syntax might look like this:

```html
    <article class="message">
      <div class="message-header">
        <slot name="header"></slot>
      </div>
      <div class="message-body">
        <slot name="body"></slot>
      </div>
    </article>
```

I have taken [Bulma](https://bulma.io/documentation/components/message/) message component, and I would like to somehow pass inject any html in those two places.

Let's start with React `<Message>` component looking like this

```javascript

const Message = props => {
  console.log(props)
  return (
    <article className="message">
      <div className="message-header">
      // Tittle should go here
      </div>
      <div className="message-body">
        // body should go here
      </div>
    </article>
  );
};
```

And the usage in `<App>`

```javascript
const App = () => {
  return (
    <main class="container">
      <Message>
        <div id="title">
          <p>Title</p>
          <div>
            <button>CTA 1 </button>
            <button> Close</button>
          </div>
        </div>
        <div id="body">Body</div>
      </Message>
    </main>
  );
};
```

When we `console.log()` our `props.children` we'll see an array containing two React elements. We start with the most naive implementation of named slots.

```javascript
const Message = props => {
  return (
    <article className="message">
      <div className="message-header">
       {props.children[0]}
      </div>
      <div className="message-body">
        {props.children[1]}
      </div>
    </article>
  );
};
```

It will work. But there are some problems with this approach:
* We render exactly everything from each node, which in our case will break the design.
* There is no visible connection in our code between usage in `<App>` and `<Message>` component
* If we pass only one child - our component will break!

To fix first one we will need something simmilar to Vue's `<template>` tag. What it could be in React's world? It's `<React.Fragment>`.

Now we will switch message component to fix all of those problems. If we have access to a `children` element. We have an access to a React element. That means we can assign `props` and use them to find what we need to fit in our slots!

Let's write a helper function for that

```javascript
  const findNamedSlot = (name, children) =>
    React.Children.toArray(children).find(
      child => child.props.name === name
    );
```

We will now fix two problems - because every React component have props, our app won't break if we won't find what we are looking for. Check out whole App here
<br>
<p class="codepen" data-height="418" data-theme-id="0" data-default-tab="js,result" data-user="khazarr" data-slug-hash="jowegX" style="height: 418px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Named Vue slot made in React - take one">
  <span>See the Pen <a href="https://codepen.io/khazarr/pen/jowegX/">
  Named Vue slot made in React - take one</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
<br>
But there is a problem. As stated in [docs](https://reactjs.org/docs/fragments.html) - `key` is the only attribute that should be passed to `React.Fragment`. Therefore it's not an ideal solution. There must be something better right?

### Compound components to the rescue!

This is exactly what we need, to have functionality similar to Vue's named slots. We need to refactor Message component to a `class` and use `static` fields for our named slots. This is better way to solve this problem.

```javascript
class Message extends React.Component {
  static Title = props => (
    <div className="message-header">{props.children}</div>
  );
  static Body = props => <div className="message-body">{props.children}</div>;

  render() {
    return <article className="message">{this.props.children}</article>;
  }
}
```

Then we can use it in our `<App>` component.

```javascript
const App = () => {
  return (
    <main className="container">
      <Message>
        <Message.Title>Title</Message.Title>
        <Message.Body>Message</Message.Body>
      </Message>
    </main>
  );
};
```

Thanks to this approach we can see very descriptive connection between our new "slots". `<Message.Title>` is a part of `<Message>`, and we can provide some content to it. What is important to remember is fact that if we **swap the order** and component will behave differently. Let's see that with codepen example:

<p class="codepen" data-height="382" data-theme-id="0" data-default-tab="js,result" data-user="khazarr" data-slug-hash="GaEzmN" style="height: 382px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Named Vue slot made in React - take two">
  <span>See the Pen <a href="https://codepen.io/khazarr/pen/GaEzmN/">
  Named Vue slot made in React - take two</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
<br>

So in fact, **compound components** gives us even more reusability/flexibility potential than named slots. But there is a little catch here. Let's extend our `<Message>` component that it will have some **state**


```javascript
class Message extends React.Component {
  
  state = {hasUserInteracted: false}

  handleInteraction = () => !this.state.hasUserInteracted && this.setState({hasUserInteracted: true})

//...
}
```
<br>
What do you think will happen if we try to access state inside our React "named slot" ?

```javascript
  static Title = props => {
    return (
      <div className="message-header">
        {this.state.hasUserInteracted ? "Thanks for reading!" : props.children}
      </div>
    );
  };
```
<br>
```
Uncaught TypeError: Cannot read property 'hasUserInteracted' of undefined
```

Do you have an idea why we cannot access state in our static property? Yup, because they are **static**.
If you are lost, please stay with me. `static` keyword comes from object oriented world. It's a `class` property, not an instance property. This means that they do not have access to `this`, but you can call them without creating new instance of a class. For example you can call `Object.assign()` without creating an Object instance first.

### So how can we fix it?

Using `props`!  But how to pass `state` as a props? We need to help ourselves with below function:
```javascript
React.cloneElement(
  element,
  [props],
  [...children]
)
```

This is the way of passing additional props to an element. And since we know, our elements are `children` of `<Message>` we need to iterate over them. But there is a catch. We cannot simply use `Array.map()` because it happens that if we have only only child React do not wrap in in array. But they provided handy method for this exact problem - `React.Children.map`.

I will create helper method for that:
```javascript
  renderChildren() {
    const {hasUserInteracted} = this.state
    
    return React.Children.map(this.props.children, child => {
      return React.cloneElement(child, {
        hasUserInteracted
      });
    });
  }
```

And you can see full working example below (hover over messagebox to toggle `hasUserInteracted` flag)
<br>
<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="js,result" data-user="khazarr" data-slug-hash="pmrEjg" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Named Vue slot made in React - take three">
  <span>See the Pen <a href="https://codepen.io/khazarr/pen/pmrEjg/">
  Named Vue slot made in React - take three</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


In my next post I will focus on how to create something analogous to Vue scoped slots in React - so stay tuned ;)



### Key takeaways

* `props.children` are pretty similar to Vue's default slot.
* If you want to create "named slots" - use compound components pattern. 
* Remember that static properties of class do not have access to `this` of a component. If you'd like to access something - use `React.cloneElement()`

---

