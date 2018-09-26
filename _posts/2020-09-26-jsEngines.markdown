---
layout: post
title:  "JavaScript Engines - 10000 Foot Overview"
date:   2018-09-26 20:18:22 +0200
categories: javascript javascript-engine
---

Nowadays JavaScript runs not only in browser but also in:
* Server
  *  Node.js
* Mobile 
  * ReactNative
  * NativeScript
* Native Apps
  * Electron

Important to notice is fact, that there is no single Engine that is widely adopted in industry, but each of the browsers vendor has it's own interpretation of it

We have:
* V8
  * Made by Chrome
  * used in Chrome/Node.js
* Chakra
  * Made by Microsoft
  * ChakraCore is OpenSource
* SpiderMonkey
  * Made by Firefox
* JavaScript Core
  * Used by Safari and React Native



Let's dive deeper into how it works. The main concept of every of this engine looks like this

![JavaScript Engine diagram](/assets/img/4/2.JPG)
  

This pipeline is a little bit different for every engine, but core idea is the same.

Engine parses code to an Abstact Syntax Tree, then intepreter can immediately get us some unoptimized results. Then code is analyzed during runtime and engine optimizes it to bytecode based on profiling and specific runtime.



One of the classical examples are fact, that in browser world, there is a lot of functions that are used only at the beginning of visit. Related to first display of an interface to user. That's why we need them fast, but not necesearly optimized. Other functions which may involve some more advanced calculations will be optimized during runtime.

---

If you want to play around with engines directly you can install them using
```
npm i -g jsvu
```
![JavaScript Engine diagram](/assets/img/4/1.JPG)

---
*rerefences:*
* https://www.youtube.com/watch?v=5nmpokoRaZI
* https://mathiasbynens.be/notes/shapes-ics



