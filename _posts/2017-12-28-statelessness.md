---
layout: page
title:  Statelessness
permalink: statelessness/
---

# {{page.title}}

Stop mutating state. We want to start writing pure functions. A pure function has no side effects, so reaching out of scope to mutate some state is not pure, and is not the functional way.

So how to manage state in our programmes? The pattern for dealing with state is to pass state around in the arguments of our functions. To illustrate this pattern I'm going to create a stateful variable that we can use to explore this idea.

```js
const Grid = (rows, cols) => {
  return new Array(rows)
    .fill(
      new Array(cols)
        .fill(null)
    )
}
/*
  Grid(3, 3)
  [
    [null, null, null],
    [null, null, null],
    [null, null, null]
  ]
*/
```

Here I'm just creating a function we can call that is going to give us our initial state, a grid of `null` values. Now we have a grid lets start to think about the logic for a game called connect4. For those of you who have never played connect4 you can find examples online; basically you have a grid like this, two players with different coloured counters. Players drop their counters into the top of the grid and they travel down the columns and rest in the first empty slot. The aim of the game is to connect 4 counters in a straight line.

It looks like we need to mutate state in this case. You might be tempted to target the array index and just say this index now equals - some value for a counter. But we don't mutate state. The idea is really simple, we take the current state, create a new version of it and pass that new state back as our output.

```js
const addCounterToGrid = (grid, col, counter) => {
  const row = pipe(
    map(nth(col)),
    findLastIndex(isNil)
  )(grid)
  return adjust(update(col, counter), row, grid)
}
```

Lets make a function that takes three arguments; the grid, the column number and the counter. In this function I want to do two things;

* find the correct row index
* Update the value at that location in the grid
* return a new version of the grid

I'm going to use pipe. First map over the rows and pull out just the value from the row that equals the column index. Now I have the column if you like, next I want to find the last null value so I'll use findLastIndex isNil. That is going to give me the row to update in the grid. I have the column index.

I'm going to use adjust; adjust takes a function who's value will become the value at a particular index in an array - the next argument in the index to adjust, finally the array to operate on. What is returned is a new array. Immutability all the way down.

The function I'm going to pass is `update` which takes two arguments an index and a value. It's last argument is an array, which `adjust` is going to pass to `update`.

Now we are set to write a test and apply the pattern I want to show you.

```js
test('you can add a few counters to the grid', () => {
  const grid1 = Grid(3, 3)
  const grid2 = addCounterToGrid(grid1, 1, "X")
  const grid3 = addCounterToGrid(grid2, 1, "O")

  expect(grid3).toEqual([
    [null, null, null],
    [null, "O", null],
    [null, "X", null]
  ])
})
```

We assign the initial state to a variable, then pass that variable to our function and assign the return value of that function to a variable, that we can then pass to the next function call - and so on. This pattern is made of functions that;

* Only act upon the arguments they receive
* Given the same inputs will always return the same outputs
* Have no side effects

I find it helpful to think about pure functions like mathematical sums.

```
2 * 8 = 16
```

Two times eight will always equal 16. The multiplier only acts on the inputs you give it, it has no side effects - for example 2 remains 2, 8 remains 8 after the operation. In this session I wanted us to recognise that there are ways of dealing with mutable state in a functional way.
