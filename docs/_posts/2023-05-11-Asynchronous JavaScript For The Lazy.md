---
layout: post
title: "Asynchronous Javascript for the lazy"
date:  2023-05-11
---


# Concurrent programming?
Asynchronous (or concurrent programming) is a form of programming that allows the code to address multiple tasks at the same time. This can practically be applied while you're waiting for an operation to complete, and want to do other things until the operation is complete. One way to visualize this is with a teapot, and a few cups of tea, as well as people drinking the tea. You can fill up someone else's tea while you wait for someone to finish their tea. You're constantly moving the teapot around, filling different cups. That example is specifically for single threaded concurrency, while multithreading could be thought of as having multiple teapots that you use to fill up multiple teacups at the same time. With one teapot, you can only fill up one cup of tea at a time. Following this metaphor, synchronous programming would be filling up a cup of tea, and then moving onto the next person, and waiting for them to finish tea. You wait for them to finish their tea, then fill it up. You might be leaving cups of tea empty for a long time, because you're sitting, waiting for someone to finish their tea. There's a few flaws in this example, but it's good enough to explain core concepts. 

Efficient asynchronous code will spend as little time as possible waiting while there's work to be done. You probably only have one teapot, use it wisely.


# The Javascript event loop
By default, Javascript is executed top to bottom, in order. 

Example:
```js
// the next line will not start execution until the previous statement completes.
console.log("called first, executing first");
console.log("called second, executing second");

// the same happens for blocks. Blocks may be called multiple times (for, while), but the 
// code inside of a block is still executed from top to bottom, and the code after a block
// will not be executed until each statement in a block has be run 

// this code will be run 3 times
for (let i = 1; i < 4; i ++) {
	console.log(`Call #${i}`);
}

console.log("called last, executing last");
```

Expected output:
```
called first, executing first
called second, executing second
1
2
3
called last, executing last
```

Concurrent JS allows your code to execute out of order.


# Event handlers
Event handling is a very basic form of concurrent Javascript. You define code that you want to run when something happens. Event driven programming is strictly well, event driven. You need to have a reliable way to determine when an event happens, and a way to call that code when the event happens. Callbacks should be implemented carefully, because badly implemented callbacks can be extremely difficult to debug.

Example: 
```js
// Callbacks are probably the most common way to implement event driven programming.
// A callback is just a function passed as an argument to another function, so that
// that function can call it at a predefined time.

// "onclick" determines when you want the callback to be called
document.getElementById("thatButton").addEventListener("onclick", () => {
console.log("Someone pressed a button!");
});
```



# Asynchronous concepts in Javascript
Concurrent Javascript has 3 primary keywords: `Promise`, `await`, and `async`.

## `Promise`
A `Promise` is an object that represents the current state of an operation. 

A promise will always be in one of three states:
- `pending`: The operation is still not complete. There's still work to be done, check back later.
- `fufilled`: The operation has completed successfully, and so I'm done working. If I have data for you, you can now access it.
- `rejected`: The operation has failed. I don't have data for you, and was unable to do what I was trying to do. 

You need to be careful when reading from a `Promise`. If you try to use the data contained in a promise before it's resolved, it can cause unexpected behavior. Race conditions can severely impact your program.

### Interacting with Promises
There's a few important ways to interact with a promise:
- `.then()`:
`.then()` can be called on a promise, and you can use it to define what you want to happen after when a promise is no longer `pending`. You can pass a callback to `.then()`, and it's executed.
```js
// this code is going to wait for 300ms, then resolve
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Called first, executing second");
  }, 300);
});

/*
 The callback passed to `.then()` takes two possible arguments. `resolve`, and `reject`
 `resolve` is passed the value contained within the `promise`, and `reject` is passed
  any errors that occur during operation. 
*/
myPromise.then((data, error) => {
	// In this case, we know the `Promise` cannot fail, but if you wanted to address
	// an error, you could do it here.
	if (error) {
		throw error;
	}
	console.log(data);
});
// while the promise is pending, the event loop continues
// once the promise is completed, the callback is executed
console.log("called second, executing first");
```
Expected output:
```
called second, executing first
called first, executing second
```

- `.catch()`
`.catch()` allows you to define a function to be called when a promise is rejected. This enables graceful error handling, and is somewhat analogous to a `try`/`catch` chain for a `Promise`.
```js
const thisPromiseWillFail = new Promise((resolve, reject) => {
  throw new Error("This didn't work");
});

// now we can gracefully catch this error
thisPromiseWillFail.catch((error) => {
  console.error(error);
});

console.log("Look mom, I'm still running!");
```

- `.finally()`
`.finally()` can be used to make interactions with chained promises more elegant. The callback accepts no arguments, and `.finally()` will return the promise it was called on without altering it. The callback passed will be called when the promise is completed, regardless of success or failure. 
```js
// It's a lot less common to define promises like this, you'll probably use
// `async` functions.
const coinFlip = new Promise((resolve, reject) => {
	// randomly resolve or reject a promise
	if (Math.random() > 0.5) {
		resolve("Heads!");
	} else {
		reject("Tails!");
	}
});

coinFlip.finally(() => {
	// This code is going to be called whether or not the promise was resolved
	// or rejected, and `.finally()` will return whatever it was passed
	console.log("Coin finished flipping");
})
.then((data, error) => {
	// we're going to handle this error with a `.catch()` call, but we could do it here
	if (error) {
		return;
	}
	// just proving that heads was actually flipped
	console.assert((data === "Heads!");
	console.log("Promise resolved, you flipped heads");
})
.catch((error) => {
	// When trying to fufil your promise, an error occured
	console.log("You flipped tails");
});
```


## `async`
If a function is defined with `async` before the function declaration, it allows the use of `await` within the function body. 

```js
// here's one way to define an async function
async function() {

}

// you can make callbacks async
const thatFunc = async () => {};
```

Async functions will always return a `Promise`, whether implicitly, or explicitly. This enables you to make use of the methods mentioned above (`then`, `catch`, `finally`). 

```js
// This function returns a promise with "hello" in it
async function hiThere() {
	return "hello";
}


// This function also returns a fufilled promise with "hello" in it
async function hello() {
	return Promise.resolve("hello")
}

// You can even return a promise without an async function
function hey() {
	return Promise.resolve("hello");
}
```

There's some minor differences in what they return with references and whatnot, they usually don't matter, but you can read up [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) if you're curious.

## `await`
`await` is the middleware between asynchronous promise wrapped values, and synchronous world of javascript. It can be used on the top level, or in `async` function declarations. When put before a function or value, `await` will *pause execution* of the surrounding `async` function until a `Promise` is either `fufilled`, or `rejected`. If the promise is rejected, than an error is thrown, and the function containing the `await` expression will appear in the stack trace. If a promise is fulfilled, than execution continues, and that line is evaluated to the value inside of the promise. If `await` is put before a value that's *not* a promise, it wraps the value in a `Promise` and evaluates it.

```js
// this function is going to return a promise that's `pending` for two seconds, 
// then resolves.
function resolveAfter2Seconds() {
	return new Promise((resolve) => {
		setTimeout(() => {
			resolve(null);
		});
	});
}


async function myAsyncFunction() {
	// this is evaluated, then we move on, like a normal function 
	console.log("called first, executing first");
	// This function returns a `Promise`. It's pending, 
	// so execution of this function is paused until it's resolved
	// because await was used.
	// in the mean time, the code after the function call is evaluated. 
	await resolveAfter2Seconds();
	// two seconds later, the promise has been resolved, and execution continues
	console.log("called second, executing third (after await)");
}

// because we aren't awaiting `myAsyncFunction` right here, the event loop will continue
// past this because it's pending.
// If we wanted to use the return value of `myAsyncFunction`, we could put an `await`
// before the call.
myAsyncFunction();
console.log("called third, executing second");
```

Output: 
```
called first, executing first
called third, executing second
called last, executing last
```

### What happens while we're waiting?
Rather than block execution of the thread, we can do other things (pour someone else's tea if you will) while we wait for the promise to be completed. While we're `await`ing a promise, we can continue evaluating stuff after the `async` function call.

Once we've run out of code to evaluate, and hit the end of our [tick](https://nodejs.dev/en/learn/understanding-processnexttick/), or one full cycle of the event loop, we go back to the top of the event loop, checking to see if any promises have been completed. If a promise has been completed, the async function resumes execution.


