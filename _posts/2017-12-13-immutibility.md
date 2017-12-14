---
layout: page
title:  Immutability
permalink: immutability/
---

# {{page.title}}

To embrace a functional style of programming you'll need to embrace the concept of immutability. Many of Javascript's primitives are not immutability by default.

Mutability in Javascript is a quality of it's core primitives; functions, arrays, and objects can all be updated in place. For arrays the following methods will mutate the array:

* pop
* push
* splice
* reverse
* sort

Immutable primitives can not be updated once they have been created. If you want an updated version you copy the original, make your changes, then return the copy, leaving the original unchanged. For example the native javascript array methods:

* map
* filter
* slice

will all return a new copy of the original array. It can be a bit confusing which array method will mutate your array and which will not. You have to learn which is which.

Lets have a quick look at the difference ramda can make with a simple example.

## Mutable primitives

```js
const a = [1,2,3,4]
const b = a.reverse()
const c = Ramda.reverse(a)

expect(b[0]).toBe(4)
expect(a[0]).toBe(1) // Fails a[0] === 4
expect(c[0]).toBe(1) // passes because ramda copies the original
```

What is the value of `a[0]` is it `1` or `4`?

It's 4.

Calling `a.reverse()` reverses the array from that point forward, we mutate the data structure. A side effect of assigning the value of 'b' is changing the value of 'a' because 'b' holds a reference to 'a'.

When I access `b[0]` am I accessing a value or a reference? This behaviour can lead to problems in more complicated apps.

```js
a.sort()
expect(b[0]).toBe(4) // fails b[0] === 1
```

It's a reference. This behaviour has lead to bugs on projects I have worked on. Immutability aims to remove the possibility of these kind of problems developing in your code.

## Diff algorithms

For anyone who has used React this pattern of immutability will be familiar to you. The original React documentation went to great lengths explaining that whilst you could update a component's state by mutation, it was much better to **not** to that. One should replace the old state with new state.

> [Do Not Modify State Directly](https://reactjs.org/docs/state-and-lifecycle.html#using-state-correctly)

<sub>from the React docs<sub>

If you modify state directly, your component will not re-render. You can't set state in the render function. Under the hood react needs the old and new states to performing it's diff algorithm that makes up part of the render function.

## Functional Reasons

Being a functional programmer means favouring Immutability for the following reasons:

* Avoid side effects
* Avoid non-deterministic values
* Purity

Purity here refers to pure functions that do not rely on external state. Pure functions only operate on the arguments they are passed, and have no side effects. As your codebase increases in size and complexity knowing that your values are stable through your implementation of immutability will lead to less bugs and help teams work more effectively together.

You can relax about implementing immutability when you are using Ramda as it comes with immutability baked in. For example.

## Updating arrays and objects

```js
const a = [1,2,3,4]
const b = a

// b[3] = 2 mutable
const update = pipe(
  adjust(always(2), 3), last
)

expect(a[3]).toBe(4)
expect(update(b)).toBe(2)
```

For objects you can do the same operation using `assoc` or `assocPath` the methods for manipulating data in arrays and objects are all in the documentation and now you understand more about their immutable behaviour.
