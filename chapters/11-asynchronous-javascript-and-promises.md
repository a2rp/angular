# 11. Asynchronous JavaScript and promises

[Back to notes index](../README.md)

| [Previous: Errors, debugging, and testing](10-errors-debugging-and-testing.md) | [Notes index](../README.md) | [Next: Fetch and REST APIs](12-fetch-and-rest-apis.md) |
|:--|:--:|--:|

## Synchronous and asynchronous work

Synchronous statements run in order. Each statement finishes before the next one begins. A slow synchronous task can block a browser page from responding.

Asynchronous operations allow other work to continue while a result is pending. Timers, user events, and network requests complete later and schedule JavaScript callbacks.

~~~js
console.log('First');

setTimeout(() => {
  console.log('Later');
}, 0);

console.log('Second');
~~~

The output is First, Second, then Later. A zero-delay timer does not run immediately; it schedules a callback for a later turn of the event loop.

## Understand the event loop

JavaScript runs a call stack of currently executing functions. The environment also manages tasks such as timers and user input, and a microtask queue for promise reactions. Once the current synchronous work finishes, queued work can run.

~~~js
console.log('Start');

setTimeout(() => console.log('Timer task'), 0);
Promise.resolve().then(() => console.log('Promise reaction'));

console.log('Finish');
~~~

The output order is Start, Finish, Promise reaction, Timer task. Promise reactions run as microtasks after the current call stack, before the next timer task.

Avoid long synchronous calculations on the browser's main thread. They prevent input and rendering from being handled until the work finishes. Web Workers can move suitable calculations off the main thread, but they use a separate message-based interface.

## A Promise represents a future result

A Promise represents an operation that may fulfill with a value or reject with a reason. Its state moves from pending to either fulfilled or rejected.

~~~js
const savedResult = new Promise((resolve) => {
  setTimeout(() => {
    resolve('Saved');
  }, 100);
});

savedResult.then((message) => {
  console.log(message);
});
~~~

The Promise executor runs immediately when the Promise is created. The timer callback and the then reaction run later.

Use then to handle a fulfilled value and catch to handle a rejection. Return a value from a then callback to pass it to the next step.

~~~js
Promise.resolve(5)
  .then((value) => value * 2)
  .then((value) => {
    console.log(value); // 10
  })
  .catch((error) => {
    console.error('The operation failed.', error);
  });
~~~

A thrown error inside a then callback rejects the next Promise in the chain. Handle the rejection at a boundary that can recover or show useful feedback.

## Use async and await

An async function always returns a Promise. The await keyword pauses that function until a Promise settles, then gives the fulfilled value or throws the rejection reason.

~~~js
function wait(milliseconds) {
  return new Promise((resolve) => {
    setTimeout(resolve, milliseconds);
  });
}

async function showProgress() {
  console.log('Working...');
  await wait(100);
  console.log('Finished.');
}

showProgress();
~~~

await pauses only the current async function. It does not block all JavaScript in the environment. Other events and tasks can continue to run.

Use try and catch to handle a rejected Promise from awaited work.

~~~js
async function loadProfile() {
  try {
    const profile = await getProfile();
    console.log(profile);
  } catch (error) {
    console.error('Could not load the profile.', error);
  }
}
~~~

getProfile represents an asynchronous function provided elsewhere. Catch an error where the code knows how to respond. If this function cannot recover, it can let the rejection reach its caller.

## Run independent work in parallel

Awaiting one request before starting another is correct when the second operation depends on the first. If operations are independent, start them together with Promise.all.

~~~js
async function loadDashboard() {
  const [profile, notifications] = await Promise.all([
    getProfile(),
    getNotifications(),
  ]);

  return { profile, notifications };
}
~~~

Promise.all fulfills when every input fulfills. If one rejects, the returned Promise rejects immediately, though the other operations may still continue.

Use Promise.allSettled when the result should include each operation's outcome even when one fails.

~~~js
async function reportDashboardResults() {
  const results = await Promise.allSettled([
    getProfile(),
    getNotifications(),
  ]);

  for (const result of results) {
    if (result.status === 'fulfilled') {
      console.log('Value:', result.value);
    } else {
      console.error('Failure:', result.reason);
    }
  }
}

reportDashboardResults();
~~~

Promise.race settles with the first input that settles. Promise.any fulfills with the first fulfilled input and rejects with an AggregateError if every input rejects. Choose based on the behavior the application needs.

## Avoid unhandled rejections

A rejected Promise should eventually be handled or deliberately returned to a caller that handles it. An unhandled rejection can become a runtime error and makes the failure difficult to trace.

~~~js
async function saveNote(note) {
  const response = await sendNote(note);
  return response;
}

saveNote({ title: 'Promises' }).catch((error) => {
  console.error('Could not save the note.', error);
});
~~~

If the caller uses await, it should catch or propagate the rejection as part of its own contract. Do not add empty catch callbacks that hide failures.

## Use timers for scheduling, not exact timing

setTimeout schedules one callback after at least the requested delay. The actual time depends on the event loop and browser scheduling.

~~~js
const timerId = setTimeout(() => {
  console.log('Reminder');
}, 500);

clearTimeout(timerId);
~~~

setInterval repeats a callback until it is cleared. For repeating asynchronous work, consider scheduling the next timeout after the current operation completes to avoid overlapping calls.

~~~js
let intervalId = setInterval(() => {
  console.log('Check for updates');
}, 5000);

clearInterval(intervalId);
~~~

Timers can be delayed when the page is busy or in the background. Do not use them as a precise clock or as proof that an animation frame has been rendered.

## Keep asynchronous flow readable

- Start independent work before awaiting the combined result.
- Use try and catch around awaited operations when there is a recovery action.
- Return Promises from helper functions instead of starting work that callers cannot observe.
- Keep loading and error state separate from the eventual data.
- Add a timeout or cancellation strategy when waiting forever is not acceptable.
- Avoid nesting callbacks when a Promise chain or async function is clearer.

## Hands-on: compare sequential and parallel work

Assume getNote and getProfile each return a Promise. First time them sequentially, then start them together. The parallel version should complete in roughly the time of the slower operation when they are independent.

~~~js
async function loadPageData() {
  console.time('sequential');
  const note = await getNote();
  const profile = await getProfile();
  console.timeEnd('sequential');

  console.time('parallel');
  const [parallelNote, parallelProfile] = await Promise.all([
    getNote(),
    getProfile(),
  ]);
  console.timeEnd('parallel');

  return { note, profile, parallelNote, parallelProfile };
}
~~~

Do not use parallel execution when one operation needs the result of another. For example, load a user ID first if the next request needs that ID.

## Notes to remember

- A Promise represents a future fulfillment or rejection.
- async functions always return Promises, and await handles a Promise inside an async function.
- Promise reactions run as microtasks after the current synchronous work.
- Promise.all is useful for independent operations that must all succeed.
- Promise.allSettled keeps each result when partial failure is acceptable.
- Handle rejections at a boundary that can respond to them.
- Timers schedule work for later; they do not guarantee an exact run time.

## References

- [MDN asynchronous JavaScript](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS)
- [MDN Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
- [MDN event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [MDN setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout)

---

| [Previous: Errors, debugging, and testing](10-errors-debugging-and-testing.md) | [Notes index](../README.md) | [Next: Fetch and REST APIs](12-fetch-and-rest-apis.md) |
|:--|:--:|--:|
