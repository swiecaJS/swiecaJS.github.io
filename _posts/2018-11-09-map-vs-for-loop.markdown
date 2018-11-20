---
layout: post
title: "Array.prototype.map() vs for loop? - thoughts after Meet.js KRK November "
date: 2018-11-09 21:49:22 +0200
categories: performance loops meetups
---

### The meetup
I have attended Meet.js KRK November where one of the talks was about Node.js performance. Krzysztof Słonka from Allegro presented his story about performance analysis
### The story
The app that was examined basically has one job to do. Convert JSON files to HTML. He described them as a worker-renders. 
The node profiler together with a flamegraph was used to determined what to optimize. Below you can see an example output from flamegraph.
![Flamegraph](/assets/img/2018-11-09/helloworld.svg)
You can manipulate with this graph to better determine 'hot' places of your application.

During their examination, they found that their app has a lot of calls to ```Array.prototype.map()``` where it is used only to apply some parameter on every element. Because ```.map()``` can take an additional parameter that was not used in their case, they replaced it with a simple ```for``` loop and gained about 11% performance boost. That's amazing! 

### My investigation

I wanted to check it myself. Let's first look at the ```.map()``` function syntax

```javascript
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
 // Return element for new_array
}[, thisArg])
```
The obligatory parameters are ```callback``` and ```currentValue```. 

Ok so let's start with a simple task of generating a new array with numbers squared.


```javascript
// setup
const forLoopMap = (array, fun) => {
 const newArray = [];
 for (let i = 0; i < array.length; i++) {
 newArray.push(fun(array[i]));
 }
 return newArray;
};

const square = (x) => {
 return x*x;
};

const numbers = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20];

// execution
numbers.map(square);
forLoopMap(numbers, square);
```

Using ```jsperf.com``` the results shown that for this simple setup, the execution time of ```Array.map()``` is **~55% slower**!

![squares results](/assets/img/2018-11-09/squares2.JPG)

---
Then I was curious about what will happen when some recursive function like Fibonacci will be used.

```javascript
const fibonacci = (num) => {
 if (num <= 1) return 1;

 return fibonacci(num - 1) + fibonacci(num - 2);
}
```
The results from both iterations over array are almost the same. When stack is overloaded with recursion calls there are no measurable benefits. 
![fib results](/assets/img/2018-11-09/fib2.JPG)

### Conclusion

It was very interesting to see in action that those small changes might actually affect your users. Krzysztof Słonka stared his presentation with famous saying that: "One second of Amazon render time could cost $1.6 Billion".

---
*resources*
* [Node.js simple profiler](https://nodejs.org/en/docs/guides/simple-profiling/)
* [Node.js CPU flamegraphs - Brendan Gregg](http://www.brendangregg.com/blog/2014-09-17/node-flame-graphs-on-linux.html)
* [JS Perf test 1 - squares](https://jsperf.com/swiecajs-map-vs-for-loop)
* [JS Perf test 2 - fibonacci](https://jsperf.com/swiecajs-map-vs-for-loop-2)
* [Amazon render time vs revenue](https://www.fastcompany.com/1825005/how-one-second-could-cost-amazon-16-billion-sales)
