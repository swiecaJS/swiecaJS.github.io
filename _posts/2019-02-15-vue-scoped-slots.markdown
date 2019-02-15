---
layout: post
title: "Vue named and scoped slots - new syntax in 2.6"
date: 2019-02-15 10:38:22 +0200
categories: javascript vue
---


Vue 2.6 gave us new syntax for scoped slots. The ```v-slot``` directive. For me that was a motivation to start using them already. 
### The basics

Let's start with regular `slots` They can be as a **blank tile** in Scrabble. You leave then inside the component, and they will become what's was passed.

```html
<NavigationButton path="/cool-offer">
  Check out this landinng page!
</NavigationButton>
```

Inside `<NavigationButton>` template you might have

```html
<router-link :to="path" tag="button" class="cta cta--primary">
  <slot></slot>
</router-link>
```

If there won't be `<slot>` in, Vue will omit things you pass. 

### More than one slot

Now stuff is getting more interesting. Imagine you've created a layout that you would like to easily reuse. Just put some content and be ready to go. But you would like to have more than one `<slot>` inside. How to distinguish between them? Using **named slots**.

```html
<MyGridLayout>
  <template v-slot:header>
    <!-- your content -->
  </template>
  <template v-slot:aside>
    <!-- your content -->
  </template>
  <template v-slot:content>
    <!-- your content -->
  </template>
  <template v-slot:footer>
    <!-- your content -->
  </template>
</MyGridLayout>
```

Inside`<MyGridLayout>` you must specify `<slot name="{name of slot}">` and you are ready to go.

### The power of scoped-slots

Scoped slots give you a very efficient way for re-using logic. Imagine you have a component responsible for rendering a list of posts. The core task - fetching data of the posts - won't change. But the *view* layer might change depending on your needs. You also always will need to handle UI change depending on network request status - pending, finished and error. But depending on *where* you will display this content you might want to use it differently. Let's create `<PostsProvider>` component.

```html
<template>
  <section>
    <slot 
      name="posts" 
      :posts="posts" 
      :isLoading="isLoading" 
      :error="error"
    ></slot>
    <slot>
  </section>
</template>

<script>
export default {
  data() {
    return {
      posts: [],
      isLoading: false,
      error: null
    }
  },
  created() {
    this.fetchPosts()
  },
  methods: {
    fetchPosts() {
      this.error = null
      this.isLoading = true
      this.isDelayElapsed = false
      getPosts()
        .then(resp => {
          this.posts = resp.data
        })
        .catch(error => {
          this.error = error
        })
        .finally(() => {
          this.isLoading = false
        })
  }
  // ...
}
</script>
```

In this component we **exposed** 3 variables - `posts`, `isLoading` and `error`. They will be accessible in *slotProps* object in parent component. All of the logic related to fetching posts is enclosed inside. Thanks to that we can easily reuse this component in multiple places of our application, for example.

```html
<PostsProvider>
  <template v-slot:posts="slotProps">
    <LoadingIndicator v-show="slotProps.isLoading && !slotProps.error"/>
    <ErrorMsg v-if="slotProps.error"/>
    <PostCardWithImage v-show="slotProps.posts.length && !slotProps.isLoading"/>
  </template>
</PostsProvider>
```

Let's introduced some short-hand syntax, you can use `#` instead of `v-slot` .Similarly as `v-bind` is `:` and `v-on` can be replaced with `@`. Also, because we exactly know what we are exposing, we can use ES6 destructuring. Let's see another example

```html
<PostsProvider>
  <template #posts="{isLoading, error, posts}">
    <ModalLoadingBox v-show="isLoading && !error"/>
    <ErrorMsg v-if="error"/>
    <PostCardWithExtendedDescription v-show="posts.length && !isLoading"/>
  </template>
</PostsProvider>
```
### Summary
As you can see, **scoped-slots** are a powerful tool. New syntax also makes use of them easier to understand and use in my opinion. I can't wait to use them on production. I made a simple demo - check it out [here](https://modest-cray-7f5a4b.netlify.com/#/)


---

_resources_

- [Vue.js docs](https://vuejs.org/v2/guide/components-slots.html)
- [Vue RFC about new syntax](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md)
- [Mateusz Rybczonek post about using scoped-slots to abstract functionality (beware, ATM old syntax)](https://css-tricks.com/using-scoped-slots-in-vue-js-to-abstract-functionality/)
- [Vue Promised - tackles the problem of awaiting, error and received state of fetching data to components using scoped-slots](https://github.com/posva/vue-promised)
