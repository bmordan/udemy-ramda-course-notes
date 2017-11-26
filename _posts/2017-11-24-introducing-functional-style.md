---
layout: page
title:  Introducing Functional Programming
permalink: introducing-functional-programming/
---

# {{page.title}}

This course is about the Ramda library, which in turn has been created to support developers programme in a functional style.

## What is the functional style?

The code we write is an abstraction. Our instructions are interpreted by our computers to do tasks. There are different ways and means to do that.

There is the imperative style for example; a build script that is a series of steps, do this then do that.

There is the declarative style where we say what values we want for example; a formula in an excel spread sheet, or a stateful variable in a React component.

Object orientated programming models concepts from our real world into the computers world. For example a programme coordinating bike hire written in an object orintated style might have in it's programme an object called `Bike` that had properties like, `docked: true`, `broken: false` or a `location`.

Functional programming says that tasks should be composed of pure functions.

> Pure functions are functions that have no side-effects. They don’t assign to any outside variables, they don’t consume input, they don’t produce output, they don’t read from or write to a database, they don’t modify the parameters they’re passed.

<sub>[Randy Coulman](http://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/)<sub>

We'll explore all of these ideas later. For now lets follow one rule.

> Functions can only receive one argument

## Addition in the functional style

Looking at the addition function, but in a functional style, is a good place to begin. Lets create a basic function that takes two numbers and adds them.

```js
function add (a, b) {
  return a + b
}
```

The challenge we face immediately is we need to pass 2 arguments, but we said we can only use one. If we pass just one our function will not work correctly.

```js
add(2) // Nan
```

In our example `b` is undefined, we try to add 2 to undefined and Javascript evaluates that to `NaN` *not a number*.

How can we solve this? Well, we could return a function that adds the first argument to something.

```js
function add (a) {
  return function (b) {
    return a + b
  }
}
```

This is the functional style. One input, one output. The input is a number, the output is a function. A function that will add whatever you passed to any consequential argument.

Lets do that.

```js
const add2 = add(2)
add2(3) // 5
```

or

```js
add(2)(3) // 5
```

It looks weird but it works because the first call with `(2)` returns a function that we can just call again with `(3)` so we are only ever passing one argument to our functions. Ramda's add function behaves like this returning a function if only one argument is passed. This is called currying and we'll look at that in the next session.
