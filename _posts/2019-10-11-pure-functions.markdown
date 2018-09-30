---
layout: post
title:  "Pure Functions - my introduction to functional programming"
date:   2018-10-11 20:18:22 +0200
categories: javascript functional-programming
---

Let start with classyfing other programming paradigms:

 - Imperative
	 - follow my commands!
 - Object-Oriented
	 - keep state and send/receive messages
 - Declarative
	 - this is what I want, I don't care how you do it, just like
	 - ```SELECT * FROM users ```
- Functional
	- pure functions !

Ok, so what is a pure function?


At the beginning let determine some rules what they can and cannot do:

 - Variables as we know then do not exist
 - We cannot share state
 - Functions should **always** return something
 - we should forget about null
 - **if** must have a pairing **else** branch
 - avoid side effects

So cobining it to a one (i hope) easy sentence
> Pure function take an input and return an output which does not affect state

Take a look at the simple example

```javascript
let apples = 2;
function  howManyApples() {
	console.log(`You have ${apples} apples`)
} 
howManyApples()
apples  =  5
howManyApples()
```

Ok so we see that this functions does not **return** anything and in addition it depends on a global variable _apples_. That's bad.
Let's rewrite it as a pure function:
```javascript
function howManyApplesPure(apples) {
	return `You have ${apples} apples`
}
console.log(howManyApplesPure(5))
```
#### Why this is good?

It is easier to test code and debug it, because in the end your functions should give explicit output to given input.
In addition if you write function which does not depent on anything, you can **reuse** it!
 
## How to write pure functions?
- Your functions cannot be time dependent

If you have specyfic time relation in your function, you cannot write a unit test

```javascript
const isNoonNow = () => new Date().getHours === 12

```
This will be true for 1 hour per day, and even it is not neccesearly bad code, it's **impure**
> your function cannot rely on random numbers
```javascript
const zeroOrOne = () => (Math.random()).toFixed(0)
```
Right even if we can test it, it is not deterministic

>cannot be I/O or network request

This is the reason why reducer/mutations cannot involve network requests in shared state libries like Vuex or Redux. 

>Start with focusing about **stateless**, which as we know might be tricky in our JS world



```javascript
const user = {
	name: 'Mike'
}
function assignSurname(user, surname)  {
	return Object.assign(user, {surname})
}
const userWithSurname = pureAssignSurname(user,  'Smith')
console.log(user) // ?
console.log(userWithSurname) // { name: 'Mike', surname: 'Smith' }
```
What do you thing will be returned? The answer is:
```{ name: 'Mike', surname: 'Smith'}```
How could it happen? We return new object, which is perfect, but our mistake with *Object.assign* led to affecting global state.
Those objects are now linked, and if you assign a new property to user it will be copied.

```javascript
user.age = 18
console.log(user)
//{ name: 'Mike', surname: 'Smith', age: 18 }
console.log(userWithSurname)
// { name: 'Mike', surname: 'Smith', age: 18 }
```

It's easy to change this function to **pure** one.
```javascript
function assignSurname(user, surname)  {
	return Object.assign({}, user, {surname})
}

const userWithSurname = pureAssignSurname(user, 'Smith')
console.log(user)
console.log(userWithSurname)
user.age = 18
console.log(user)
console.log(userWithSurname)
```
Console output:
```
{ name: 'Mike' }
{ name: 'Mike', surname: 'Smith' }
{ name: 'Mike', age: 18 }
{ name: 'Mike', surname: 'Smith' }
```
> avoid mutability, do not change values reassign them
```javascript
const favouriteAnimals = ['cats','dogs','rats']
// let's remove rats
favouriteAnimals.pop() //ALERT! SIDE EFFECTS!

const actualFavouriteAnimals = favouriteAnimals
	.filter(animal => animal !== 'rats')
```