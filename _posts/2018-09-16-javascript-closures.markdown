---
layout: post
title:  "JavaScript closures"
date:   2018-09-16 12:08:22 +0200
categories: javascript closures es6
---

### What, Why and How? 
In JavaScript closure is a basic way to enable data privacy.
We start with formal definitions:

> A _closure_ is the combination of a function and the lexical environment within which that function was declared.

>A closure is a function having access to the parent scope, even after the parent function has closed.

So in other words, this is a way of accessing another function scope which is not global. A straightforward definition could be
> A closure is a way of making private variables for function, which are NOT availible in global scope, but they are still availible for your use even after funcion is closed

Ok, that may seem scary at the beginning, but let’s move to examples.

```javascript
// simple example
const  secretKeeper  =  (secret)  =>  {
	return  ()  =>  `your secret is ${secret}`
}  	
const  mySecret=  secretKeeper('28')
console.log(mySecret())  //will log 'your secret is 28'
```
As you can see we have a function that returns a function. We cannot directly access secret, but the value of a secret is stored after function evaluated. That’s great! We have everything we need.

Ok, so now you think how this may be useful. This is a great pattern for factory functions, which have similar purpose and you do not need to think about leaking variables to global, or that you may override them.

```javascript
const  employee  =  ({name,  payout,  raiseFactor})  =>  {
	const  startingSalary  =  payout
	let  workedYears  =  0
		return  ({isRaiseGiven})  =>  {
			if  (isRaiseGiven)  {
				payout  *=  raiseFactor
			}
			return  `Employee name: ${name}, 
			current salary: ${payout.toFixed(2)}k$, 
			starting salary: ${startingSalary.toFixed(2)}k$,
			years worked: ${++workedYears}, 
			raiseGivenThisYear: ${isRaiseGiven}`
		}
}

  
const  Adam  =  employee({
  name:  'Adam',
  payout:  80,
  raiseFactor:  1.1
})

const  Bob  =  employee({
  name:  'Bob',
  payout:  100,
  raiseFactor:  1.03
})
 
console.log(Adam({isRaiseGiven:  false}))
/* Employee name: Adam, 
current salary: 80.00k$, 
starting salary: 80.00k$  
years worked: 1, 
raiseGivenThisYear: false 
*/
console.log(Bob({isRaiseGiven:  true}))
/* Employee name: Bob, 
current salary: 103.00k$, 
starting salary: 100.00k$  
years worked: 1, 
raiseGivenThisYear: true 
*/

console.log(Bob({isRaiseGiven:  true}))
/* Employee name: Adam, 
current salary: 88.00k$, 
starting salary: 80.00k$  
years worked: 2, 
raiseGivenThisYear: true 
*/
console.log(Adam({isRaiseGiven:  true}))
/* Employee name: Bob, 
current salary: 106.09k$, 
starting salary: 100.00k$  
years worked: 2, 
raiseGivenThisYear: true 
*/

```
We created factory functions for an employee, and what is great we can create as many employees as we need, and the internal variables (raiseFactor, name, payout) cannot be changed, and in addition, you cannot override them by mistake. This is huge!

Also what is great we can provide an additional parameter when calling employee, end it may behave as we need, without worrying about global scope!