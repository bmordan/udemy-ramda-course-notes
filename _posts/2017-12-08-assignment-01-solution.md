---
layout: page
title:  Assignment 1 Solution
permalink: assignment-01-solution/
published: true
---

# {{page.title}}

My solution is just one of many possible ways to do this task. Let me talk you through my thinking.

The first thing I'll need to do is extract my values from the data. Then with my two values I'll perform a subtraction calculation taking the 'sunrise' time from the 'sunset' time. The times look like they are *time in seconds from epoch*.

So first thing is get the two values I want to operate on 'sunrise' and 'sunset'. I can see they are included in the 'sys' property so I start by accessing that property with **prop**.

**prop** When given an object returns the named property.

Then I want to do the same with two properties. I'm going to use **props** which is going to get 'sunrise' and 'sunset'.

**props** When given an object and array of keys will return those properties.

I am now happy that I have my two values.

Ramda's subtract function is curried by default, and it matters which order I pass these values to subtract. Ramda's subtract works like:

> Subtract from this value (first argument), this value.
> Subtract from 10, 8 = 2

I need to pass the higher value to subtract first of all. I have an array and want to end up with a single value - so sounds like a job for **reduce**.

But problems when I tried to do this. In Ramda's reduce the iterator function has the accumulator as the first argument, then the iterating value. In which case **subtract** would receive 0, then the first item in the list.

> Subtract from 0 the sunrise time -> -sunrise

Its all wrong. I want to switch the order of the arguments, the value first then the accumulator. In the docs for **reduce** it says **reduceRight** works like that so lets use **reduceRight**.

**reduceRight** is reduce turned on it's head. The arguments to the accumulator are flipped and it starts iterating over your array from last to first.

> Subtract from the sunrise, 0 -> sunrise

Nearly. Now we are using reduceRight to feed in the higher value as the first item we have to also **reverse** the array of values. Which is not a problem for **reverse** that does exactly that.

At this point the calculation is complete. We have a value, problem is it's in seconds not hours, so I want to divide by 60 a couple of times, so I make a function *divideBy60* and use that to turn seconds into minutes, and minutes into hours. Final touch to remove any floating points, Math.floor.

If you ended up with a different solution thats fine. I hope you learnt some new things about Ramda and the functional programming style. To be honest this is my refined solution.

You can now tackle basic problems using the functional programming style. Right?
