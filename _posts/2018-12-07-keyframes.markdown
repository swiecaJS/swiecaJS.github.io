---
layout: post
title: "Keyframe animations basics"
date: 2018-12-08 11:19:22 +0200
categories: css animation
---

There is no doubt that animations are an important part of web design. Computers are used to boolean values. Either something is true or false, right? But for human beings, this is not so simple. We feel strange if things appear out of nowhere. I hope you remember memes with cats being afraid of cucumbers. (If not check links in resources xD). Having said that, in my opinion, the most important use of animation is to present **change** in a controlled manner.

Look at this example below, try to hover on those boxes

<p data-height="300" data-theme-id="0" data-slug-hash="jXNGjQ" data-default-tab="result" data-user="khazarr" data-pen-title="for post about animation" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/jXNGjQ/">for post about animation</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

<br>
I above example I have used ```transiton``` property on ```opacity```. But what if we want some more cool action on hoover? Let's use keyframes! The **main difference** between transition and animation is fact, that transition has only two states - start and end. 

We need to define the name and what will happen from start (0%) to end of our animation (100%). In our case, we will have three steps
Let's enhance opacity example. We create ```form-appear``` animation.

```css
  @keyframes form-appear {
    0% {
      transform: scale(0) translateY(100px);
      opacity: 0;
    }

    50% {
      transform: scale(1.05) translateY(-15px);
      opacity: 1;
    }

    100% {
      transform: scale(1) translateY(0px);
      opacity: 1;
    }
  }
```

To trigger animation we need to use ```animation``` keyword.

```css
.example {
  animation: form-appear 1s ease;
}
```

I have added also another animation for every form element

```css
  .example > * {
    opacity: 0;
    animation: slideX 1s ease;
    animation-delay: .5s;
    animation-fill-mode: forwards;
  }

  @keyframes slideX {
    0% {
      opacity: 0;
      transform: scaleX(0);
    }
    100% {
      opacity: 1;
      transform: scaleX(1);
    }
  }
```

The most important parameter here is ```animation-fill-mode``` which tells the browser to keep the final state of animation. Otherwise, after the animation ends it will go back to the initial state - in our case ```opacity: 0```.

<p data-height="332" data-theme-id="0" data-slug-hash="bObYaz" data-default-tab="html,result" data-user="khazarr" data-pen-title="post about animation #2" class="codepen">See the Pen <a href="https://codepen.io/khazarr/pen/bObYaz/">post about animation #2</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


---
*resources*
* [Short and to the point YT tutorial](https://www.youtube.com/watch?v=iUSC_7hcPPo)
* [Animista - greate place with examples](http://animista.net/)
* [Cats afraid of cucumbers on YT](https://www.youtube.com/watch?v=pXv44YL_Gio)
* [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes)




