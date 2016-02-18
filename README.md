# Callback Hell

## Overview

You might be familiar with callback from browser JavaScript. In a nutshell, they are functions which we pass as arguments to be invoked later. For example, a callback of an XHR call might involve rendering the response from a RESTful API server. That's browser JavaScript, but Node we use callback often too to ensure the sequence of the execution.

Let's say you have to implement three HTTP requests to a third-party service (like Facebook or Google), but they need to be performed in a sequential order, because the second and third request use information from the response of the first request (e.g., API token).  This is the pseudo code:

```
callOne(function(error, data1) {
    callTwo(function(error, data2) {
        callThree(function(error, data3) {
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

You might end up with something called pyramid of doom or callback hell. The code is very hard to read and maintain. People who have to work on this code after you might start to hate you!

Th `readdir` example is from <http://callbackhell.com> (yes, there is even a website for Callback Hell!).

This lesson will cover the ways to combat callback hell.

## Objectives

1. Observe a callback hell script as a possible complication of the callback async pattern
1. Comprehend possible ways to combat callback hell with named functions 
1. Comprehend possible ways to combat callback hell with modules


## Using Named Function to Mitigate Callback Hell

The easiest way to reduce the level of indentation and thus reduce the callback hell is to use named function. Named functions are functions with names or variables which values are functions.

```js
function processResponse1(error, data1) {
  ...
}

callOne(processResponse1)
```

or 

```js
var processResponse1 = function() {
  ...
}

callOne(processResponse1)
```

The difference is that the named function will be hoisted, meaning its definition can be anywhere in the file while the variable function's definition must precede the usage of it. Another difference between named functions and variables is that named function will have names in the stack trace (the error message for debugging) so they are better for trouble shooting and debugging..

Now where did the other two request go? They are inside of the `processResponse1`:

```js
callOne(processResponse1)

function processResponse1(error, data1) {
  callTwo(processResponse2)
}

function processResponse2(error, data1) {
  callThere(processResponse3)
}

function processResponse3(error, data1) {
  ...
}
```

Do you find this code easier to understand? We too! But that's now it. We can take it a step further.

Note: `callOne`, `callTwo`, etc. are just pseudo code for making HTTP requests. In real-life example you would substitute them for real functions using libraries like `http`, `request` or `superagent`. For now it's not really important what those functions do as long as they are asynchronous and use callbacks.

## Using Modules to Mitigate Callback Hell

Imagine having to work on a file with 10 named functions as listed in the previous section. You have 5 co-workers working on the same file. Each callback/response is 50 lines of code. That's insane! Of course it's better than having them in the closures (i.e., anonymous function definitions), but we know how to use modules so let's apply them by making each callback a separate module:

```js
var processResponse1 = require('./response1.js')
callOne(processResponse1)
```

This is the `response1.js` file:

```js
var processResponse2 = require('./response2.js')
module.exports = function processResponse1(error, data1) {
  callTwo(processResponse2)
}
```

This is the `response2.js` file:

```
var processResponse3 = require('./response3.js')
module.exports = function processResponse2(error, data2) {
  callThere(processResponse3)
}


This is the `response3.js` file:

```js
module.exports = function processResponse3(error, data3) {
  ...
}
```


So now your 5 co-workers will love you even more because each piece of logic will be in its own file (you can easily separate files into different folder by providing paths to the `require`). Another added benefit of modular approach is that you and your team can write unit tests for each module. Having unit tests greatly increases robustness of the system and the velocity of development, because developers can make changes without being afraid of unintentionally breaking existing functionality (if they break the tests will catch that!).

## Resources

1. [Callback Hell, where bad code goes to rest](http://callbackhell.com)
1. []()
1. []()


---

<a href='https://learn.co/lessons/node-non-blocking-callback' data-visibility='hidden'>View this lesson on Learn.co</a>
