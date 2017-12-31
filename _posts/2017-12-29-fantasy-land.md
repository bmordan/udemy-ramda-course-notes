---
layout: page
title: Fantasy Land Specification
permalink: fantasy-land/
---

# {{page.title}}

As you explore the Ramda documentation you are going to happen upon the term 'Fantasy Land'. There is no actual 'Fantasy Land'. Instead 'Fantasy Land' is a reference to a specification we can use to standardise our functions. Standardised functions will play nicely together enabling us to compose them without problems.

In the 'Fantasy Land' there is the idea of 'types' of common algebraic structures.

> An algebra is a set of values, a set of operators that it is closed under and some laws it must obey.

Let's explore this a bit more, it is not as fantastical as is seems. We know Arrays. An Array is a common algebraic structure. In fact it is actually a class type of 'Functor'. 'Functors' have to implement a '.map' method which Arrays do out of the box.

```js
[2].map(n => console.log(n))
// 2
```

What is a 'Functor'? You can think about it as a wrapper around your value. In the example above our value is the Number 2, and the array structure is like a box around it. This is the opening to another paradigm in functional programming.

```js
2 + 3
// 5
```

2 plus 3 is 5, easy these are just values. I make a curried function called add3 that will add 3 to a value. I wrap the value 2 and 3 in the 'Functor' algebraic structure (an array) and try to do the same.

```js
[2] + [3]
// "23"
```

Ha! it is actually the String "23". You've got to love javascript! What I want is `[5]` the result of the two values, that shares the same wrapper. I can use a curried function and the `.map` method exposed by the functor.

```js
const add3 = add(3)
[2].map(add3)
// [5]
```

The interesting behaviour is this.

```js
[].map(add3)
// []
```

If there is no value, the function is not applied, your just handed back an empty box.

'Functors' allow you to apply functions to their containing value via the `.map` method. You get your result back wrapped in a 'Functor'.

How about a wrapped function applied to a wrapped value? For that we need to progress our 'Functor' into an 'Apply'.

A value which has an 'Apply' must provide an `ap` method. The `ap` method takes one argument:

```js
a.ap(b)
```

* b must be an Apply of a function
* If b does not represent a function, the behaviour of ap is unspecified.
* b must be same Apply as a.
* a must be an Apply of any value
* ap must apply the function in Apply b to the value in Apply a
* No parts of return value of that function should be checked.
* The Apply returned by ap must be the same as a and b

```js
const wrappedValue = [2]
const wrappedFn = [add3]
Array.prototype.ap = function (wrappedFn) {
  return wrappedFn.map(fn => fn(this[0]))
}
expect(wrappedValue.ap(wrappedFn)).toEqual([5])
```

There is more. Lets develop our Arrays to also become 'Applicative'. A value which has an Applicative must provide an `of` function on its type representative. The `of` function takes one argument:

* of must provide a value of the same Applicative
* No parts of a should be checked

```js
Array.prototype.of = function (val) {
  return new Array(val)
}
```

Finally let us look at one more type `Chain`.

A value which has a Chain must provide a chain method. The chain method takes one argument.

```js
[].chain(fn)
```

* fn must be a function which returns a value
* fn must return a value of the same Chain
* chain must return a value of the same Chain

With Chain we will have to have some logic that checks the value returned by the passed in function is truthy. If its falsy we just return `this` so we can carry on chaining.

```js
Array.prototype.chain = function (fn) {
  return !this.map(fn)[0] ? this : this.map(fn)
}
```

Now we can do some thing rad. Chaining means we can apply the same function over and over, if it blows up in our face with an undefined, everything carries on our chain is not broken.

```js
[3].chain(add3)
   .chain(add3)
   .chain(() => undefined)
   .chain(add3)

// [12]
```

In summary we have taken the 'Fantasy Land' spec and enhanced our Array with:

* .ap apply
* .of applicative
* .chain

We have actually created a 'Monad'. A 'Monad' is a 'Functor' that also implements; 'Apply', 'Applicative' and 'Chain'. Its an algebraic structure.

A 'Monad' usually implements a couple of other helper methods too.

* just
* nothing

We can use `just` to pull the value out of the Monad, and `nothing` to tell us if we have a value in our Monad at all.

```js
Array.prototype.just = function () {
  return this[0]
}
Array.prototype.nothing = function () {
  return !this[0]
}
```

The 'Monad' is a pattern you will come across in different functional programming languages. It gives us a robust way to deal with `undefined` and `null` values in our programs when the output of functions maybe one thing, or maybe nothing.
