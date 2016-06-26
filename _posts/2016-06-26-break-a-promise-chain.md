---
title: Map a stream of promises to the latest
layout: post
categories: blog development
tags:
- javascript
- js
- promise
---

# Map a stream of promises to the latest

Let's assume that I have a function that creates a promise. When this function is called consecutively, I want the last promise to succeed and all others to reject if they have not already been resolved.

This sounds similar to `flatMapLatest` from RxJS. The way I solved this problem is as follows:


```js
// function that creates a promise
// it resolves after 1 second.
function makePromise() {
    return new Promise((res, rej) => {
        setTimeout(() => {
            res();
        }, 1000);
    });
}

function latest(fn) {
    // we use function call count
    // to check if it is the last function call
    // when the promise resolves
    let count = 0;

    // don't use arrow function to have the same
    // `this` context as the original function call
    return function() {
        let currCount = ++count;
        const countChecker = (x) => {
            // reject promise if this is not the last
            // function call
            if(currCount !== count) {
                return Promise.reject('Chain broken');
            }

            // if we don't reject, return the same resolved
            // value
            return x;
        };
        return fn.apply(this, arguments)
            .then(countChecker);
    };
}

// example usage:
const newMakePromise = latest(makePromise);

const success = () => console.log('Success');
const fail = () => console.log('Fail');

newMakePromise().then(success, fail);
setTimeout(() => newMakePromise().then(success, fail), 500);

// Output:
// Fail
// Success
```

I can put the promise returned by `newMakePromise` between any two promises to break a promise chain:

```
const chainBreaker = latest(makePromise);

const refreshData = () => {
    return makeRequest().then(store).then(chainBreaker).then(display);
};

/* The following will `makeRequest` and `store` twice but `display`
   only the last one
*/
refreshData();
refreshData();

```

