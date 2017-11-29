---
layout: page
title:  Composing Functions
permalink: composing-functions/
---

# {{page.title}}

In this session we are going to start composing functions. This is were functional programming really began to make sense to me.

In our previous example where we were looking at curried functions, we hashed password strings using a curried function.

Seeing as our heads are into that lets carry on and make things a little more interesting. Hashing every user's password with the same salt is not so great. Each new hashed password we should create with a new salt. Then we should store the hashed password string and the salt together.

When the time comes to check that a password string matches the password we have stored for that user, we will:

* recover the hashed password and salt combination
* split the salt away from the stored hash
* bind the stored salt to our hashing function
* create a callback that checks the stored key matches the hashed key

This is a procedure that we can create with a series of small functions composed together. Ramda has a suite of compose functions:

* compose
* composeK
* composeP
* pipe
* pipeK
* pipeP

Lets create a simple composed function with a function that adds 5 to a number and a function that squares it's input. You pass a value into the pipe, then the output of the first function is the input of the next until you return a function or final value.

```js
const addFive = add(__, 5)
const sqr = (n) => n * n
// compose innermost right to left
expect(compose(addFive, sqr)(0)).toBe(5)
expect(compose(sqr, addFive)(0)).toBe(25)
// pipe reverses innermost left to right
expect(pipe(sqr, addFive)(0)).toBe(5)
expect(pipe(addFive, sqr)(0)).toBe(25)
```

Compose works right to left, pipe works left to right.

In our pipe we need to:

* recover the hashed password and salt combination
* split the salt away from the stored hash
* bind the stored salt to our hashing function
* create a callback that checks the stored key matches the hashed key

```js
const username = 'cat_lover'
const password = 'cats12'
const iterations = 1000
const len = 24
const digest = 'sha512'

const keyGen = curry(pbkdf2)(password, __, iterations, len, digest)

const getHashedPassword = always('0409757bb520dfc434eb9252b1176082324af3134f71e247someuniquestringtoactasasalt')

const cb = (err, key, storedKey) => {
  expect(err).toBeFalsy()
  expect(key.toString('hex')).toEqual(storedKey)
  done()
}
const generateMatch = (storedKey, salt) => keyGen(salt)(partialRight(cb, [storedKey]))

const verify = pipe(
  getHashedPassword,
  splitAt(48),
  converge(generateMatch, [head, last])
)

verify('cat_lover')
```

I'm imagining an endpoint that has been hit with a username and password. We can curry our hashing function with password and our default config. Then we would lookup the stored hashedPassword/salt with the username in a datastore, but here we are just going to make a function that always returns our string.

**always** will always return the value you supply it with.

Next we split the returned string.

**splitAt** will split a string or array at a particular index. Seeing as we know the length of our generated key we can break the string and end up with an array of two values the storedKey and the salt.

Now we want to do different things with our 2 values so I'm going to use converge.

**converge** will pass it's input to the functions supplied in an array, the output of all those functions is passed to the converge function.

**head** returns the first item in an array
**last** returns the last item in an array

Our converge function is going to receive the storedKey and salt. We are going to add the salt value to our curried hashing function. Now that hashing function is primed with the user's password and their unique salt when we call it we can expect the value of key in the callback to equal the storedKey we have just retrieved.

This is a bit tricky because we want the value of storedKey to be available in our callback. The arity of the callback is a list of 2 arguments, an error and a key. Lets add the storedKey to those arguments with partialRight.

**partialRight** takes a function and an array of values. Use partial to add those values to the arguments of the function you passed. We can add storedKey using partialRight, so our callback will actual be called with:

* error
* key,
* storedKey

If we just used partial it would add the arguments to the beginning of the list so we'd get

* storedKey
* error
* key

To complete our pipe, we can add the callback to our curried hashing function and then call it.

The passwords match.

As you can see here we can trigger sequences of tasks using compose or pipe. Our logic is not trapped into complex methods, but just assembled with small functions.

## Tap for debugging

I want to show you a useful tool when working with pipelines.

**tap** tap will call a function with the supplied input, and then return the input as it's output. It is like a pass through function. Use it with console.log to inspect inputs/outputs in your pipe. For us the first input is the username;

* Then it is the retrieved storedKey and salt
* Next an array of storedKey and salt
* Then those values are passed to our final function

Use tap to inspect your pipeline. Now you are ready to tackle a coding problem using Ramda.
