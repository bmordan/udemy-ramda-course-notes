---
layout: page
title:  Assignment 2 Solution
permalink: assignment-02-solution/
published: true
---

# {{page.title}}

There are is no one correct way to solve this problem. Here is my walk through for the final assignment.

For inputs I am being passed the grid (an array of arrays), and the reference of the last played counter. The output is an array of arrays, the references of 4 connected counters.

I am going to need to lookup references around the lastPlayed reference. I will have to deal with null and undefined references. I also will have to lookup in two related directions.

Here are the Ramda functions I have used:

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
  last
} = require('ramda')
```

I made a list of simple transform functions that will transform a reference. So you can pass a reference to `up` and it will return you a transformed reference to the cell above the original reference in the grid.

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

This will not always work when you look these references up on the grid. For example if you are on the right edge of the grid and you call `right` the new reference will be to a cell which does not exist and will be undefined. So I wanted a way to deal with null and undefined values. Here is my `Maybe` monad, and a function that uses it.

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

I need to be looking for cells in two related directions. For example `upRight` and `downLeft` makes a diagonal line in which we need to find 4 connected counters. So I just structure them in an array that I can reduce over.

```js
const transFns = [
  [up, down],
  [left, right],
  [downLeft, upRight],
  [upLeft, downRight]
]
```

I can feel the solution might involve some recursive functions, and some reducing. It will not be a direct mapping. When I have found 4 connected counters, I want to stop, the game is won. `reduceWhile` seemed a great fit for that.

`reduceWhile` takes a predicate function that it called before each iteration, if it returns false it will quit and return the current value of the accumulator.

```js
const connect4 = (arr) => gt(4, arr.length)
```

The `connect4` function tests that 4 is greater than the accumulator array's length. When connect4 is false it breaks the reduce.

Lots of small bits now to start putting them together. The main `reduceWhile` is going to reduce over the transform functions performing lookups. The idea is we'll start with the lastPlay reference. Any winning array of references will have this reference as one of its values, so its our seed value.

The programme can't see the whole grid like we can. It's harder to find patterns. I am going to look in each direction one cell at a time. If the next cell is null or undefined, we can stop and then change direction.

If we find a match, then we'll add it to our result and go again. I needed a recursive function that I could call again and again until I have exhausted that direction. Here is my lookup function.

```js
const lookup = curry((grid, counter, prevRef, acc, transFn) => {
  const newRef = transFn(prevRef)
  const maybe = maybeCounter(grid, newRef)
  if (maybe.nothing()) return acc
  if (maybe.just() !== counter) return acc
  return lookup(grid, counter, newRef, append(newRef, acc), transFn)
})
```

It's curried so I can call it with different transform functions. My iterator function is called with a pair of related transform functions like `up` and `down`. It tries to find a match only returning when it does not.

After a lookup I'm left with an array that has the references of matches to the left and matches to the right.

```js
[ [ [ 4, 4 ], [ 5, 3 ] ], [ [ 2, 6 ] ] ]
```

I finally combine these two arrays, with the original reference to end up with a possible winning combination.

```js
[ [ 2, 6 ], [ 3, 5 ], [ 4, 4 ], [ 5, 3 ] ]
```

Here is the iterating function.

```js
const iterator = (grid, counter, lastPlay, transFns) => {
  const found = map(lookup(grid, counter, lastPlay, []), transFns)
  const connections = pipe(
    flatten,
    splitEvery(2)
  )([last(found), lastPlay, head(found)])
  return connect4(connections) ? lastPlay : connections
}
```

The last line is a bit clumsy `reduceWhile` is going to do this same logic too. If we have not found a winning connection of 4 counters in a particular direction, I want to start the next round of lookups with a clean slate if you like. So I test for connect 4 and if I don't have it, we just result the accumulator to be the origin reference, or the last played reference.

```js
const yourFn = (grid, lastPlay) => {
  const counter = maybeCounter(grid, lastPlay).just()
  return reduceWhile(
    connect4,
    partial(iterator, [grid, counter]),
    lastPlay,
    transFns
  )
}
```

The entry function is the `reduceWhile` function I have mentioned. I'm partially applying the grid and the lastPlayed counter so these values are available in the iterator.

This is enough to make the test pass, the algorithm has found the references for the connected counters. The main goal of the final assignment has been to have you use some of the functions provided by Ramda, combine it with the pieces of functional programming theory we have looked at, and to write some code to solve a problem.

I hope you have enjoyed the course. Please rate it, leave your comments and tell your friends and colleagues about it.
