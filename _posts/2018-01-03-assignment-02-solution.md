---
layout: page
title:  Assignment 2 Solution
permalink: assignment-02-solution/
published: false
---

# {{page.title}}

There is no one correct way to solve this problem. Here is my walk through for the final assignment solution. I used these Ramda functions;

```js
const {
  gt,
  reduceWhile,
  partial,
  adjust,
  inc,
  dec,
  pipe,
  map,
  curry,
  append,
  flatten,
  splitEvery,
  head,
  last,
  tap
} = require('ramda')
```

## My Approach

As I thought about solving the problem there were a couple of programming patterns I felt would be a good fit. The first was recursion, a function that would keep calling itself until it didn't. I imagined such a function looking up adjacent references, accumulating those references and then calling itself again if it found a matching counter. That pattern of accumulating references made me also think of a reduce function. So I was expecting to use some recursion and a reduce.

I visualised my algorithm working something like this. I start at one reference in the grid. Then I look in each direction one cell at a time. If the next cell is null or undefined, I can stop and then change direction.

If I find a match, then I'll add it to my results, and call the same function again and again until I have exhausted that direction. I liked the idea of looking in related directions, for example left and right, then concatenating the matching results. The moment that concatenated result's array has a length of 4, the game has been won, and I'm left with the winning references all lined up in the right order.

## Transform Functions

I started simple. To look up adjacent cells I needed to derived the references for those cells from a given grid reference. I made a list of pure functions that will transform a reference. So you can pass a reference to `up` and it will return you a transformed reference to the cell above.

```js
const up = adjust(dec, 0)
const down = adjust(inc, 0)
const left = adjust(dec, 1)
const right = adjust(inc, 1)
const upLeft = pipe(adjust(dec, 0), adjust(dec, 1))
const upRight = pipe(adjust(dec, 0), adjust(inc, 1))
const downLeft = pipe(adjust(inc, 0), adjust(dec, 1))
const downRight = pipe(adjust(inc, 0), adjust(inc, 1))

up([2, 3])
// [1, 3]
```

The transformed reference will not always work when you use it to lookup a counter on the grid. For example if you are on the right edge of the grid and you call `right` the new reference will be to a cell which does not exist, and will be undefined. So I wanted a way to deal with null and undefined values. I created a `Maybe` monad, and a function that uses it.

```js
const Maybe = function (val) { this.val = val }
Maybe.of = function (val) { return new Maybe(val) }
Maybe.prototype.nothing = function () { return (this.val === null || this.val === undefined) }
Maybe.prototype.just = function () { return this.val }

const maybeCounter = (grid, ref) => {
  const [rowInd, colInd] = ref
  const maybe = Maybe.of(grid[rowInd])
  return maybe.nothing() ? maybe : Maybe.of(grid[rowInd][colInd])
}
```

With `maybeCounter` I can supply it with the grid and a reference and receive a `Maybe` in return. The condition of the `Maybe` would decide if I keep looking in that direction or give up.

I need to be looking for matching cells in two related directions. For example `upRight` and `downLeft` makes a diagonal line in which we need to find 4 connected counters. So I just structure my transform functions in related pairs.

```js
const transFns = [
  [up, down],
  [left, right],
  [downLeft, upRight],
  [upLeft, downRight]
]
```

## Reducing

Now to start building the execution flow. My entry function is going to use a `reduceWhile`. `reduceWhile` takes a predicate function that it calls before each iteration. If the predicate function returns false it will quit and return the current value of the accumulator. The accumulator here will be the array of connected counters. My predicate function looks like this. If 4 is greater than the length of the accumulator keep looking.

```js
const keepLookingConnect4 = (arr) => gt(4, arr.length)
```

My `reduceWhile` also needs an iterating function, as well as the seed value (which will be the lastPlayed reference) and a list. The list I'm reducing over is the list of transform functions, arranged in related directional pairs. My entry function now looks like this;

```js
const yourFn = (grid, lastPlay) => {
  const counter = maybeCounter(grid, lastPlay).just()
  return reduceWhile(
    keepLookingConnect4,
    partial(iterator, [grid, counter]),
    lastPlay,
    transFns
  )
}
```

In a normal reduce, the iterating function receives two arguments (acc, value) I want to add the grid and the counter as arguments too, so I use a partial. Now my iterating function will get passed;

* the grid
* the counter
* the lastPlayed reference
* a pair of directional transform functions

Here is my iterating function.

```js
const iterator = (grid, counter, lastPlay, transFns) => {
  const found = map(lookup(grid, counter, lastPlay, []), transFns)
  const connections = pipe(
    flatten,
    splitEvery(2)
  )([last(found), lastPlay, head(found)])
  return keepLookingConnect4(connections) ? lastPlay : connections
}
```

In the iterator `found` is an array of two values that result from mapping over the pair of directional transform functions. Those values in `found` are arrays of references found in the two directions. `connections` is a concatenation of the original reference with references we found in the two directions.

If we have a winning array of 4 references then I return that array - reduceWhile will terminate and we'll have a result. If it is less that 4 then I return the original reference, so we can start a fresh round of lookups in the next direction.

## The lookup function

Here is my lookup function.

```js
const lookup = curry((grid, counter, prevRef, acc, transFn) => {
  const newRef = transFn(prevRef)
  const maybe = maybeCounter(grid, newRef)
  if (maybe.nothing()) return acc
  if (maybe.just() !== counter) return acc
  return lookup(grid, counter, newRef, append(newRef, acc), transFn)
})
```

It's curried so I can call it with different transform functions in a map. It does the following work;

* creates a new reference by applying a transform function
* tries to look that cell up on the grid
* if there is nothing it returns
* if there is a counter that does not match it returns
* if there is a match it calls the lookup again with new arguments

The recursive call to lookup has two altered arguments the value of 'prevRef', and the 'acc' or accumulator now has any found references added to it.

When this is all put together it is enough to make the test pass, the algorithm has a way to find 4 connected counters in any direction.

The main goal of the final assignment has been to have you use some of the functions provided by Ramda, combine it with the pieces of functional programming theory we have looked at, and to write some code to solve a problem.

I hope you have enjoyed the course. Please rate it, leave your comments and tell your friends and colleagues about it.
