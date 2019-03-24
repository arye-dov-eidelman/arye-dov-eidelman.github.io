---
layout: post
title:      "6 reasons you're confused by promises"
date:       2019-03-01 13:56:57 -0500
permalink:  6_reasons_youre_confused_by_promises
---


```javascript
new Promise(()=>{
    // Do a thing, possibly async, then…

    // Wait what? What exactly does 'possibly' mean here?
    throw "I'm confused.";
  })
  .catch(reason=>{
    if (youAreStillReading){
      console.log("Here are some reasons why promises are confusing:");
    } else {
      console.log("It looks like async/await stole the show.");
      console.log("We're probably better off that way.");
    }
  });
```

1. logging a fulfilled promise in Google Chrome logs a state of "resolved" instead of "fulfilled" as per the spec [<sup>[promise resolve functions]</sup>](https://www.ecma-international.org/ecma-262/#sec-promise-resolve-functions) [<sup>[fulfillpromise]</sup>](https://www.ecma-international.org/ecma-262/#sec-fulfillpromise). This is an implementation detail and shouldn't affect any code.

1. in theory promises should be broken with the [break<sup>2</sup>](https://www.merriam-webster.com/dictionary/break) method. But because `break` is a keyword in JavaScript (like most languages) the terms [reject](https://www.merriam-webster.com/dictionary/reject) and rejected are used instead. Even though reject is jeneraly only used in reference to the opinions of __others__.

1. Some terminology is based on a high-level description of promises where it's an analogy to promises between people where they can be `fulfilled` or ~~broken~~ `rejected`. While some terminology is based off a low-level description of promises as a placeholder for the eventual result of an asynchronous operation where it will `resolve` to a value and `then` the value can be used for future operations.

1. Many tutorials don't mention that __then__ can take __2__ functions `onFulfilled` and `onRejected`. Without knowing this the term then seems odd as `then` infers that it always happens after the previous promise resolves, which is only the case if it is passed two functions.

    ```javascript
    new Promise((resolve, reject)=>reject("error message"))
      .then(
        // callback for if it fulfills
        result=>console.log(result),

        // callback for if it rejects
        error=>console.log(error)
      );

    //> error message
    ```

1. Because promises are known for being asynchronous many tutorials disregard the fact that the function passed to the promise constructor runs immediately not asynchronously!

    <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise#Creating_a_Promise>

    > ```javascript
    > const myFirstPromise = new Promise((resolve, reject) => {
    >   // do something asynchronous which eventually calls either:
    >   //
    >   //   resolve(someValue); // fulfilled
    >   // or
    >   //   reject("failure reason"); // rejected
    > });
    > ```
    >
    > To provide a function with promise functionality, simply have it return a promise:
    >
    > ```javascript
    > function myAsyncFunction(url) {
    >   return new Promise((resolve, reject) => {
    >     const xhr = new XMLHttpRequest();
    >     xhr.open("GET", url);
    >     xhr.onload = () => resolve(xhr.responseText);
    >     xhr.onerror = () => reject(xhr.statusText);
    >     xhr.send();
    >   });
    > }
    > ```

    <https://www.sitepoint.com/overview-javascript-promises/#theapi>

    > ```javascript
    > const promise = new Promise((resolve, reject) => {
    >   //asynchronous code goes here
    > });
    > ```

    <https://developers.google.com/web/fundamentals/primers/promises#whats-all-the-fuss-about#promises_arrive_in_javascript>

    > Do something within the callback, perhaps async

    Perhaps, as in if you're calling a different async function here otherwise it isn't async?

1. Why is this even the case? I thought promises are about running asynchronous operations. [<sup>[mdn]</sup>](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) [<sup>[aplus Promise spec]</sup>](https://github.com/promises-aplus/promises-spec)

    Shouldn't the following two examples be the same?

    ```javascript
    myFunction = (v) => console.log(`Is example ${v} running async?`)

    example1 = new Promise(resolve => resolve()).then(()=>{myFunction(1)})
    example2 = new Promise(resolve => {myFunction(2) })

    console.log("Above logs were run synchronously");

    // Is example 2 running async? // no
    // Above logs were run synchronously
    // Is example 1 running async? // yes
    ```

    Instead in the first one myFunction runs asynchronously, while in the second one it runs synchronously.

It isn't just me who has issues with the terminology, [google's developer documentation](https://developers.google.com/web/fundamentals/primers/promises#promise-terminology) makes note of the confusing terminology.

  > ## Promise terminology
  > [Domenic Denicola](https://twitter.com/domenic) proof read the first draft of this article and graded me "F" for terminology. He put me in detention, forced me to copy out States and Fates 100 times, and wrote a worried letter to my parents. Despite that, I still get a lot of the terminology mixed up, but here are the basics:
  >
  > A promise can be:
  >
  > - fulfilled - The action relating to the promise succeeded
  > - rejected - The action relating to the promise failed
  > - pending - Hasn't fulfilled or rejected yet
  > - settled - Has fulfilled or rejected
  >
  > [The spec](https://www.ecma-international.org/ecma-262/#sec-promise-objects) also uses the term thenable to describe an object that is promise-like, in that it has a then method. This term reminds me of ex-England Football Manager [Terry Venables](https://en.wikipedia.org/wiki/Terry_Venables) so I'll be using it as little as possible.

Some of this information above is based on [this discussion from 2012](https://github.com/promises-aplus/promises-spec/issues/5) about what terminology should be used. Let's just say some opinions we're rejected but no promises were broken 😉.

It's interesting to note that the new async await syntax solves a lot of the issues and hides a lot of these terms. Now Instead of `reject`ing, we exclusively `throw` errors and instead of chaining an `onFulfill` function. We `await` for the previous value to `fulfill` before continuing. We also use the standard try...catch...finally syntax.

