---
layout: page
title:  Point-free
permalink: point-free/
---

# {{page.title}}

You will hear functional programmers talk about 'point-free'. Now it's time for you to join there ranks. 'Point-free' is a compositional style where we de-couple our functions from the data they operate on.

Let me explain with a simple example. In our assignment we pass a data object into our function. Say we want to construct a summary sentence from that data.

> Today in London you can expect broken clouds

We can construct that sentence from picking out a couple of properties from the data. I could do some string interpolation using the data.

```js
(data) => `Today in ${data.name} you can expect ${data.weather[0].description}`
```

That is NOT point free, because I refer to the data object directly in my function. Lets have a go at this problem and create a function thats 'point-free'.

Lets start with a function called 'summarySentence'. We are going to compose a few functions in a pipe and I'm starting by tapping the input.

```js
const summarySentence = pipe(
  tap(console.log)
)

expect(summarySentence(data)).toBe('Today in London you can expect broken clouds')
```

From our data we want to pick off the two properties that we need to create our summary sentence, so lets use __pick__ to get 'name' and 'weather'.

Now let's have a look at an awesome utility called __evolve__, this takes an object and allows you to evolve all or some of the properties of that object by applying a transformation function. I'll show you want I mean.

```js
const transformsWeather = {
  weather: pipe(
    map(prop('description')),
    join(' ')
  )
}
```

Here I'm creating a transform function that will operate on the weather value which is an array. Lets __map__ over the array and return only the description values. Then turn those values into a string by joining them all with a space between them using __join__.

__evolve__ then returns an object, and I can use __props__ to return an array of strings ready to be parsed into a sentence.

I am just having so much fun using Ramda that the next steps are maybe a bit over the top, but please indulge me. I'm going to use __zip__. __zip__ creates a new list out of the two supplied by pairing up equally-positioned items from both lists. I just can't resist doing this for our string interpolation.

Nearly there. Now it's just a map and __join__ party to get strings out of the array structures.

```js
const transformsWeather = {
  weather: pipe(
    map(prop('description')),
    join(' ')
  )
}
const summary = pipe(
  pick(['name', 'weather']),
  evolve(transformsWeather),
  props(['name', 'weather']),
  zip(['Today in', 'you can expect']),
  map(join(' ')),
  join(' ')
)
```

I never reference the data in this collection of functions. Sometimes 'point-free' is called 'tacit' the definition of which is very helpful:

> tacit - understood without being openly expressed

The data is operated on by our pipe of functions, but not explicitly referenced by it. So our atomic functions remain generic, wonderfully stupid and simple. As you compose functions together, think about:

* passing the data as the final argument
* creating small functions that are generic
* avoid referencing your data explicitly

Follow these ideas and you'll find yourself naturally writing code in a 'point-free' style.
