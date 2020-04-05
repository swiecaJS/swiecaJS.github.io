---
layout: post
title: "VS Code - TS type check in Javascript files"
date: 2020-04-05 11:38:22 +0200
categories: vscode tools typescript javascript
---

There is no dobut that TypeScript's type checking can prevent bugs even before you run the code. It's an amazing feature. But did you know that you can turn it on in regular `.js` files?

### The problem

Imagine you have written code like this. At first glance you might not see any problem.

![](/assets/img/vs-code-ts/1.png)

But actually `localstorage` should be **camelCased**. 

### Seamless solution

Just add the following at the top of the file:
```js
// @ts-check
```

And it's done! 🚀

![](/assets/img/vs-code-ts/2.png)

Amazing! Isn't it?

You can also set it up globally using below settings:
```js
{
  "javascript.implicitProjectConfig.checkJs": true
}
```

---
*resources*
* [VS Code Frontend Masters course](https://frontendmasters.com/courses/customize-vs-code/)










