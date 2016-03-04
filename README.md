# Callback Hell

## Overview

We've used callbacks in previous units to create an easier workflow and schedule events to happen in the future, as opposed to our code being executed in a linear order. While callbacks are incredibly useful, they can get pretty complicated pretty quickly.

Let's say you have to implement three HTTP requests to a third-party service (like Facebook or Google), but they need to be performed in a sequential order, because the second and third request use information from the response of the first request (e.g., API token).  We can't delay code the way we do in traditional synchronous style, so you end up with nested callbacks to ensure the right order of execution. This is the pseudo code:

```js
callOne({...}, function(error, data1) {
    callTwo(data1, function(error, data2) {
        callThree(data2, function(error, data3) {
        ...
        })
    })
})
```

Remember, this is asynchronous world so we cannot type `callOne` then `callTwo` on a new line. We need to use callback and with callbacks (the way they are written in the example above) comes indentation.

Now add on top of that a few callbacks to perform something else essential like opening a database connection and starting a server. Look at this:


```js
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
}
```

You might end up with something called the nested approach, pyramid of doom or callback hell. This code is very hard to read and maintain. People who have to work on this code after you might start to hate you!

The `readdir` example is from <http://callbackhell.com> (yes, there is even a website for Callback Hell!).

This lesson will cover the ways to combat callback hell.

## Objectives

1. Explain that a callback hell script is a complication of the callback async pattern
1. Identify a callback hell script
1. Implement possible ways to combat callback hell with named functions
1. Implement possible ways to combat callback hell with modules


## Using A Named Function to Mitigate Callback Hell

The easiest way to reduce the level of indentation and thus reduce callback hell is to use named functions. Named functions are functions with names or variables which values are functions.

```js
function processResponse1(error, data1) {
  ...
}

callOne({...}, processResponse1)
```

or

```js
var processResponse1 = function(error, data1) {
  ...
}

callOne({...}, processResponse1)
```

The difference is that the named function will be hoisted, meaning its definition can be anywhere in the file while the variable function's definition must precede the usage of it. Another difference between named functions and variables is that named function will have names in the stack trace (the error message for debugging) so they are better for trouble shooting and debugging..

Now where did the other two request go? They are inside of the `processResponse1`:

```js
callOne({...}, processResponse1)

function processResponse1(error, data1) {
  callTwo(data1, processResponse2)
}

function processResponse2(error, data2) {
  callThere(data2, processResponse3)
}

function processResponse3(error, data1) {
  ...
}
```

This code is considerably easier to read, but that's not all we can do to improve it. We can take it a step further.

Note: `callOne`, `callTwo`, etc. are just pseudo code for making HTTP requests. In real-life example you would substitute them for real functions using libraries like `http`, `request` or `superagent`. For now it's not really important what those functions do as long as they are asynchronous and use callbacks.

## Using Modules to Mitigate Callback Hell

Imagine having to work on a file with 10 named functions as listed in the previous section. You have 5 co-workers working on the same file. Each callback/response is 50 lines of code. That's insane! Of course it's better than having them in the closures (i.e., anonymous function definitions), but we know how to use modules so let's apply them by making each callback a separate module:

```js
var processResponse1 = require('./response1.js')
callOne({...}, processResponse1)
```

This is the `response1.js` file:

```js
var processResponse2 = require('./response2.js')
module.exports = function processResponse1(error, data1) {
  callTwo(data1, processResponse2)
}
```

This is the `response2.js` file:

```js
var processResponse3 = require('./response3.js')
module.exports = function processResponse2(error, data2) {
  callThere(data2, processResponse3)
}
```

This is the `response3.js` file:

```js
module.exports = function processResponse3(error, data3) {
  ...
}
```


So now your 5 co-workers will love you even more because each piece of logic will be in its own file (you can easily separate files into different folder by providing paths to the `require`). Another added benefit of modular approach is that you and your team can write unit tests for each module. Having unit tests greatly increases robustness of the system and the velocity of development, because developers can make changes without being afraid of unintentionally breaking existing functionality (if they break the tests will catch that!).

## Callback Arguments Convention

Typically callbacks follow `error, result` convention where `error` is an error object (null is there's no errors) and `result` which is what you would expect from a function. Because this is just a convention, libraries and frameworks don't have to follow it. That's why you must read the usage for each library before attempting to use it's methods.

The names you use in the callbacks for arguments such as `error` or `result` don't really matter. The best practice is to make them meaningful to avoid confusion (`error` is better to read than `err` or `e` which might mean an event to other developers working on this code). What matters in the callback declarations is **order** of arguments.

Lastly, the callback function is typically the last argument. It's another convention, so it's not enforced by the Node platform in any way. It's just the best practice to follow. Again, you will often see different names for callbacks such as `cb`, `done`, `next`, `end`, and our favorite `callback` (because it's easier to read and understand).

## Resources

1. [Callback Hell, where bad code goes to rest](http://callbackhell.com)
2. [Concurrency vs. Parallelism](http://stackoverflow.com/questions/1050222/concurrency-vs-parallelism-what-is-the-difference#1050257)
3. [Avoiding Callback Hell in Node.js](http://stackabuse.com/avoiding-callback-hell-in-node-js)
4. [Managing Node.js Callback Hell with Promises, Generators and Other Approaches](https://strongloop.com/strongblog/node-js-callback-hell-promises-generators)
5. [Redemption from Callback Hell Video](https://www.youtube.com/watch?v=hf1T_AONQJU)

---

<a href='https://learn.co/lessons/node-non-blocking-callback' data-visibility='hidden'>View this lesson on Learn.co</a>
