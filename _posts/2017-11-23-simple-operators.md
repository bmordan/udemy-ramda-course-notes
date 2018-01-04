---
layout: page
title:  Simple Operators
permalink: simple-operators/
---

# Simple Operators

Thank you for choosing this course. My name is Bernard I’m a full stack web developer. Most of my years have been spent creating apps in JavaScript. When I was introduced to functional programming in JavaScript with the Ramda library I felt my programming world expanding and new vistas opening up.

I am really looking forward to taking you through the sessions that comprise this course. Each session has a video for you to watch. Each session has a link to the transcript of the session. You might find yourself watching a session a couple of times. That’s totally fine. I have packed them.

I my mind I see you watching the video then creating the examples for yourself, maybe playing or experimenting a little bit too. Each session is building toward one of two ‘Assignments’.

The ‘Assignments’ pose you coding problems to solve. You can solve them however you like. If the test passes you’ve done it. You can think about the ‘Assignments’ as your playground.

In this first session we are going to just get ourself setup, and look at some simple operators.

You'll need 'nodejs' installed on your machine and there are course notes to help you do this. I'm going to make a folder and install two dependencies into it.

Jest is a testing framework developed at 'Facebook'. You create test files with a .test extention. so my first file needs to end `.test.js`.

```sh
touch simple-operators.test.js
```

Now I'm going to update the npm package.json test command to be `jest --watchAll` that will watch my test files and run the tests on save.

In the Ramda documentation they assign the library to a singleton named `R`. I don't really like that, I find it anoying to write `R.` everywhere. So you'll see me importing just the functions I want like this.

```js
const {
  add,
  concat
} = require('ramda')
```

So lets write a test.

```js
test('import ramda', () => {
  expect(add(2, 3)).toBe(5)
  expect(concat('Hello ', 'World')).toEqual('Hello World')
})
```

nice.
