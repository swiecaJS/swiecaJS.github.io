---
layout: post
title: "Algorithm challange you might face on the front-end side"
date: 2019-04-14 09:38:22 +0200
categories: javascript vue
---


Just a regular day in your developer job. You are checking your tasks for today. A client want's a gallery view, with buttons. One big central image, and couples of images at the bottom. User can interact with gallery using two buttons. One will show next images, second will show previous ones. An overview of functionallity below.

Ok, let's face it. If you'll need some gallery with preview you'd probably `npm install` some third party package. Then you'll check the docs, import it somwhere in your code. And after 20 mins you could proudly mark task as done.
But you might miss all the **fun** that is happening underneath!

Let's assume, that all of the logic regarding showing images will happen on the **client side**. You will get just an array of images from backend. So actually you have to implement an algorithm for *array traversing*. We'll enapsulate this logic in JavaScript class. Thank's to that, our solution will be framework agnostic.

Let's start modelling our interface:
```javascript
class ImageGallery {
 constructor(imagesArray) {
   this.imagesArray = imagesArray
   this.currentIndex = 0
 }

 getMainImage() {
   return this.imagesArray[this.currentIndex]
 }

 nextImage () {
   // ?
 }
}
```

In *constructor* we provide imagesArray, and we set **currentIndex** to 0. At first, we would like to implement method for just changing image to next one. So what conditions should outr **nextImage** method has? We definitely want to increment **currentIndex**, but on the other hand we don't want to be outside of array.

```javascript
nextImage() {
    if (this.currentIndex < this.imagesArray.length) {
      this.currentIndex = this.currentIndex + 1;
    } else {
      this.currentIndex = 0;
    }
}
```

First idea might be something like this. If our current index is greater than array length, return 0 (which is first object of array). But is it ok?

Let's try with array with two items ['a', 'b']

```javascript
const gallery = new ImageGallery(["a", "b");
// current index = 0
gallery.nextImage();
// this.currentIndex < this.imagesArray.length
// 0 < 2
// we increment our index


gallery.nextImage();
// current index = 1
// this.currentIndex < this.imagesArray.length
// 1 < 2
// we increment our index again
// current index = 2 WHOOPS!
```

Having in mind that array start's at 0 in JS, we just get out of our **imagesArray** scope. And our call to **getMainImage()** will return *undefined*.
We need to substract 1 from total array length to make it work!


```javascript
  nextImage() {
    if (this.currentIndex < this.imagesArray.length - 1) {
      this.currentIndex = this.currentIndex + 1;
    } else {
      this.currentIndex = 0;
    }
  }
```

Ok, now we would like to have ability to go to the previous image. The logic will be simmilar.
* If current index is greater than 0, substract 1
* If our current index is equal to 0, we need to the last element, therefore it will be equal to imagesArray lenght - 1.


```javascript
  prevImage() {
    if (this.currentIndex > 0) {
      this.currentIndex = this.currentIndex - 1;
    } else {
      this.currentIndex = this.imagesArray.length - 1;
    }
```

Ok so we have basic functionality, now it's time to look where it's really interesting. We would like to have 2 images at the bottom of the main image, and display next images. To imagine that look at this example

```javascript
const images = ['a','b','c',]

// main image - a
// bottom - b,c

// main image - b
// bottom - c,a

//main image c
//bottom - a,b
```

Can you see that we are actually traversing an array with subarray?
Let's take another look, focusing only at the bottom row. This time three items.

```javscript
/*

array - abcdefg

a[bcd]efg
ab[cde]fg
abc[def]g
abde[fgh]
a]bdef[gh
ab]cdefg[h
[abc]defgh
*/
```

Do you see the pattern? We need to present the *next* three ones. Having in mind that our array ends somwhere. Let's start by extracting our method for getting next index. Also we need to obtain gallery indexes and then when we have indexes we can get gallery values. We start with gallery size fixed to two.

```javascript
  getNextIndex(currentValue) {
    if (currentValue < this.imagesArray.length - 1) {
      return currentValue + 1;
    } else {
      return 0;
    }
  }

  nextImage() {
    this.currentIndex = this.getNextIndex(this.currentIndex);
  }

    getGalleryIndexes() {
    return [
      this.getNextIndex(this.currentIndex),
      this.getNextIndex(this.currentIndex + 1)
    ];
  }

  getGallery() {
    return this.getGalleryIndexes().map(index => this.imagesArray[index]);
  }

```

Ok. Now let's focus on how our **getNextIndex**  handles new situation. Can you spot the problem?

```javascript
const gallery = new ImageGallery(["a", "b", "c", "d"]);
console.table(gallery);

```
![](/assets/img/js-image-gallery/console-table-1st-example-0.png)

That's our starting poing, let's select next image three times and see how our gallery handles that


![](/assets/img/js-image-gallery/console-table-1st-example-1.png)
![](/assets/img/js-image-gallery/console-table-1st-example-2.png)
![](/assets/img/js-image-gallery/console-table-1st-example-3.png)

Can you see that? When currentIndex is equal to 3 -which is the last element. The gallery outputs two time 'a', 'a'. And why it's that?

Focus on our logic:
 - we obtain gallery by calling *getGalleryIndexes* first

 ```javascript

  // current index = 3

   getGalleryIndexes() {
    return [
      this.getNextIndex(3),
      this.getNextIndex(3 + 1)
    ];
  }
 ```

 - we can spot that the second image has a problem. How **getNextIndex** will handle this case?

 ```javascript

 // currentValue 4
 // imagesArray.length - 3
   getNextIndex(currentValue) {
    if (currentValue < this.imagesArray.length - 1) {
      return currentValue + 1;
    } else {
      return 0;
    }
  }

  // therfore

  if (4 < 4 -1 ) {
    // Not here
  } else {
    return 0;
    // returns first element of an array
  }
 ```


Ok we now exactly where is the problem. How to solve it?

We need to add third case to **getNextIndex**! But how it will look like?

- if currentIndex is greater do something

Let's try to create that using values from above

```javascript

if (currentValue > this.imagesArray.length - 1) {
// ?
} 

// let's try doing the same as we incrementing this value

if (currentValue > this.imagesArray.length - 1) {
// ?
return currentValue + 1 
} 

// our explicit case
// currentValue 4
// imagesArray.length - 4

 // we are looking for NEXT index, therefore in mind we can quickly calculate that this will be equal to 1
 
 // This case happens when currentValue is greater than array length


 if (4 > 4 - 1) {
   return 4 + 1 
 }

 // we get 5, which is outside of the array.
 
 // But we know that we are doing circular traversal. Therfore if we get outside, we need to go back to the beginning.

 // Therefore we can substract array length!

  if (4 > 4 - 1) {
   return 4 + 1 - 4
 }

 // we get our deisred 1 !

```

Below updated version of **getNextIndex**.

```javascript

   getNextIndex(currentValue) {
      if (currentValue < this.imagesArray.length - 1) {
        return currentValue + 1;
      } else if (currentValue > this.imagesArray.length - 1) {
        return currentValue + 1 - this.imagesArray.length;
      } else {
        return 0;
      }
  }

```

Now we have all elements to create it!

Working example below:

<p class="codepen" data-height="396" data-theme-id="0" data-default-tab="result" data-user="khazarr" data-slug-hash="wZrvez" style="height: 396px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="gallery example">
  <span>See the Pen <a href="https://codepen.io/khazarr/pen/wZrvez/">
  gallery example</a> by Karol Świeca (<a href="https://codepen.io/khazarr">@khazarr</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


Now a task for you, can you enhance this, to take gallery size in constructor?

---

