---
layout: page
title:  Curried Functions
permalink: curried-functions/
---

# {{page.title}}

A curried function has transformed the number of arguments ('arity') it can receive from many to one. The curried function keeps returning a function that you can call with the next argument until you are done and get your final result.

The 'add' function in Ramda is curried. We can call 'add' with just one argument and we get back a function that we call with the next argument until the function returns the final result.

```js
const add2 = add(2)
add2(3) // 5
```

or

```js
add(2)(3) // 5
```

## An example

Lets explore this idea of curried functions using an aspect of nodejs 'crypto' API. Let's import 'pbkdf2' - password-based-key-derivation-function-2 (I'm going to call it our hashing function). This is a one-way hashing function that you might use to hash a password string before adding it to a datastore. The function require's 6 arguments:

* a password string
* a salt
* the number of iterations
* the final key length
* the name of a digest to use
* a callback function to receive the key

The number of iterations is the number of iterations the hashing algorithm will use to create the key. More is best but the more iterations you set the longer it takes to compute.

Example's of digests you can use are: ```['md4', 'md5', 'sha1', 'sha256', 'sha512', 'whirlpool']```

Lets curry this function with Ramda

```js
const keyGen = curry(pbkdf2)
```

Now we can call this function with just one argument at a time. For example we can bake in the user's password string.

```js
const withPassword = keyGen('cats12')
```

We have not created the key yet instead we have a function with the password string bound to it. We can pass in the remaining arguments one by one sticking to our functional rule.

```js
const password = 'cats12'
const salt = 'someuniquestringtoactasasalt'
const iterations = 1000
const len = 24
const digest = 'sha512'
const genKeyWithPassword = keyGen(password)(salt)(iterations)(len)(digest)
```

Now we have a function ready to create a crypto hash of our password string. All we pass to it is a function that will receive the key.

```js
genKeyWithPassword((err, hash) => {
  console.log(err || hash.toString('hex')) // 0409757bb520dfc434eb9252b1176082324af3134f71e247
})
```

What does currying really do for us here? Well it gives us some flexibility. We don't need all the arguments at the same time and in the same place to create our key. This becomes really powerful when we start using placeholders for one, or more, of the arguments. Ramda has a placeholder function called `underscore underscore` lets use it.

Having created and stored our user's password string in the form of a hash, later we'll want to verify that the user's supplied password string matches the one we have stored.

Here we can use Ramda's placeholder `__` function in lieu of the password string. This means we can create a 'verify' function which we can call later with a password string.

```js
const verify = keyGen(__, salt, iterations, len, digest)
const create = verify(password)
const stored = '0409757bb520dfc434eb9252b1176082324af3134f71e247'
const cb = bind(done, (err, hash) => {
  expect(hash.toString('hex')).toEqual(stored)
  done()
})
//usage
create(cb)
verify('cats12')(cb)
```

Having curried our hashing function we can easily make a `create` function to make and store our key, and a `verify` function to check that passwords match. This is simplified example, my aim here to show how programming in a functional style leads you to create more reusable functions. Next we need to look at composing functions.

## digests for pbkdf2

```
[ 'DSA',
  'DSA-SHA',
  'DSA-SHA1',
  'DSA-SHA1-old',
  'RSA-MD4',
  'RSA-MD5',
  'RSA-MDC2',
  'RSA-RIPEMD160',
  'RSA-SHA',
  'RSA-SHA1',
  'RSA-SHA1-2',
  'RSA-SHA224',
  'RSA-SHA256',
  'RSA-SHA384',
  'RSA-SHA512',
  'dsaEncryption',
  'dsaWithSHA',
  'dsaWithSHA1',
  'dss1',
  'ecdsa-with-SHA1',
  'md4',
  'md4WithRSAEncryption',
  'md5',
  'md5WithRSAEncryption',
  'mdc2',
  'mdc2WithRSA',
  'ripemd',
  'ripemd160',
  'ripemd160WithRSA',
  'rmd160',
  'sha',
  'sha1',
  'sha1WithRSAEncryption',
  'sha224',
  'sha224WithRSAEncryption',
  'sha256',
  'sha256WithRSAEncryption',
  'sha384',
  'sha384WithRSAEncryption',
  'sha512',
  'sha512WithRSAEncryption',
  'shaWithRSAEncryption',
  'ssl2-md5',
  'ssl3-md5',
  'ssl3-sha1',
  'whirlpool' ]
```
