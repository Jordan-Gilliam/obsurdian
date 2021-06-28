# React 18
## ✅ Out-of-the-box improvements
React 18 will include out-of-the-box improvements:

-   [Automatic batching for fewer renders](https://github.com/reactwg/react-18/discussions/21)
-   [SSR support for Suspense](https://github.com/reactwg/react-18/discussions/22)
-   [Fixes for Suspense behavior quirks](https://github.com/reactwg/react-18/discussions/7)

## ✅ Concurrent features

React 18 will be the first React release adding opt-in support for concurrent features such as:

-   **[`startTransition`](https://github.com/reactwg/react-18/discussions/41)**: lets you keep the UI responsive during an expensive state transition.
-   **`useDeferredValue`** lets you defer updating the less important parts of the screen.
-   **`<SuspenseList>`**: lets you coordinate the order in which the loading indicators appear.
-   **[Streaming SSR with selective hydration](https://github.com/reactwg/react-18/discussions/37)**: lets your app load and become interactive faster.

You can start using these features in a small part of your tree without enabling Strict Mode for your entire app.

_Note: if you're familiar with concurrent "mode", see: [What happened to concurrent "mode"](https://github.com/reactwg/react-18/discussions/64)._

## ✅ What about Suspense for data fetching?

The React 18 release includes the foundational work related to `<Suspense>`, but it will likely not include the recommended data fetching solution yet. A complete solution would need to include [Server Components](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) and a [built-in Cache](https://github.com/reactwg/react-18/discussions/25), and these projects are still in progress. We expect to see them complete during the 18.x timeline, but not necessarily in the 18.0 initial release.

For full details about `<Suspense>` in React 18, see [this answer](https://github.com/reactwg/react-18/discussions/47#discussioncomment-847004).


____
____

# New API: createRoot

> [SOURCE](https://github.com/reactwg/react-18/discussions/5)

React 18 ships two root APIs, which we call the Legacy Root API and the New Root API.

-   **Legacy root API**: This is the existing API called with `ReactDOM.render`. This creates a root running in “legacy” mode, which works exactly the same as React 17. Before release, we will add a warning to this API indicating that it’s deprecated and to switch to the New Root API.
-   **New root API**: The new Root API is called with `ReactDOM.createRoot`. This creates a root running in React 18, which adds all of the improvements of React 18 and allows you to use concurrent features. This will be the root API moving forward.

## What is a root?

In React, a “root” is a pointer to the top-level data structure that React uses to track a tree to render. In the legacy API, the root was opaque to the user because we attached it to the DOM element, and accessed it through the DOM node, never exposing it to the user:

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Initial render.
ReactDOM.render(<App tab="home" />, container);

// During an update, React would access
// the root of the DOM element.
ReactDOM.render(<App tab="profile" />, container);

```

In the New Root API, the caller creates a root and then calls render on it:

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Create a root.
const root = ReactDOM.createRoot(container);

// Initial render: Render an element to the root.
root.render(<App tab="home" />);

// During an update, there's no need to pass the container again.
root.render(<App tab="profile" />);
```

## What are the differences?

There are a few reasons we changed the API.

First, this fixes some of the ergonomics of the API when running updates. As shown above, in the legacy API, you need to continue to pass the container into render, even though it never changes. This also mean that we don’t need to store the root on the DOM node, though we still do that today.

Second, this change allows us to remove the `hydrate` method and replace with with an option on the root; and remove the render callback, which does not make sense in a world with partial hydration.

## What about hydration?

We’ve moved the `hydrate` function to a `hydrateRoot` API.

Before:

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Render with hydration.
ReactDOM.hydrate(<App tab="home" />, container);
```

After:

```jsx
import * as ReactDOM from 'react-dom';

import App from 'App';

const container = document.getElementById('app');

// Create *and* render a root with hydration.
const root = ReactDOM.hydrateRoot(container, <App tab="home" />);
// Unlike with createRoot, you don't need a separate root.render() call here
```

Note that unlike `createRoot`, **`hydrateRoot` accepts the initial JSX as the second argument.** This is because the initial client render is special and needs to match with the server tree.

If you want to update a root again _after_ hydration, you can save it to a variable, just like with `createRoot`, and call `root.render()` later:

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Create *and* render a root with hydration.
const root = ReactDOM.hydrateRoot(container, <App tab="home" />);

// You can later update it.
root.render(<App tab="profile" />);
```

## What about the render callback?

In the legacy root API, you could pass `render` a callback that is called after the component is rendered or updated:

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

ReactDOM.render(container, <App tab="home" />, function() {
  // Called after inital render or any update.
  console.log('rendered').
});
```
In the New Root API, we’ve removed this callback.

With partial hydration and progressive SSR, the timing for this callback would not match what the user expects. To avoid the confusion moving forward, we recommend using `requestIdleCallback`, `setTimeout`, or a ref callback on the root.

So instead of this:

```jsx
import * as ReactDOM from 'react-dom';

function App() {
  return (
    <div>
      <h1>Hello World</h1>
    </div>
  );
}

const rootElement = document.getElementById("root");

ReactDOM.render(<App />, rootElement, () => console.log("renderered"));
```
You can do this:

```jsx
import * as ReactDOM from 'react-dom';

function App({ callback }) {
  // Callback will be called when the div is first created.
  return (
    <div ref={callback}>
      <h1>Hello World</h1>
    </div>
  );
}

const rootElement = document.getElementById("root");

const root = ReactDOM.createRoot(rootElement);
root.render(<App callback={() => console.log("renderered")} />);
```

See [codesandbox](https://codesandbox.io/s/cold-pine-eyr62?file=/src/index.js).

## Why ship both?

We’re shipping the legacy API in React 18 for two reasons:

-   **Smooth upgrade**: We want to avoid user upgrading to React 18 and immediately seeing a crash. Instead, we’re adding a warning to the legacy root API that recommends using the new API.
-   **Experimentation:** Some apps may choose to run a production experiment to compare the legacy root with the new root which includes performance improvements out of the box. By including both, it’s easier to run experiments.

----
----
# New Feature: Automatic batching for fewer renders

>[SOURCE](https://github.com/reactwg/react-18/discussions/21)

React 18 adds out-of-the-box performance improvements by doing more batching by default, removing the need to manually batch updates in application or library code. This post will explain what batching is, how it previously worked, and what has changed.

> Note: this is an in-depth feature that we don’t expect most users to need to think about. However, it may be relevant to educators and library developers.

## What is batching?

Batching is when React **groups multiple state updates into a single re-render** for better performance.

For example, if you have two state updates inside of the same click event, React has always batched these into one re-render. If you run the following code, you’ll see that every time you click, React only performs a single render although you set the state twice:

```js
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount(c => c + 1); // Does not re-render yet
    setFlag(f => !f); // Does not re-render yet
    // React will only re-render once at the end (that's batching!)
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
-   ✅ [Demo: React 17 batches inside event handlers](https://codesandbox.io/s/spring-water-929i6?file=/src/index.js). (Notice one render per click in the console.)

This is great for performance because it avoids unnecessary re-renders. It also prevents your component from rendering “half-finished” states where only one state variable was updated, which may cause bugs. This might remind you of how a restaurant waiter doesn’t run to the kitchen when you choose the first dish, but waits for you to finish your order.

However, React wasn’t consistent about when it batches updates. For example, if you need to fetch data, and then update the state in the `handleClick` above, then React would _not_ batch the updates, and perform two independent updates.

This is because React used to only batch updates _during_ a browser event (like click), but here we’re updating the state _after_ the event has already been handled (in fetch callback):

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 17 and earlier does NOT batch these because
      // they run *after* the event in a callback, not *during* it
      setCount(c => c + 1); // Causes a re-render
      setFlag(f => !f); // Causes a re-render
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
-   🟡 [Demo: React 17 does NOT batch outside event handlers](https://codesandbox.io/s/trusting-khayyam-cn5ct?file=/src/index.js). (Notice two renders per click in the console.)

Until React 18, we only batched updates during the React event handlers. Updates inside of promises, setTimeout, native event handlers, or any other event were not batched in React by default.

## What is automatic batching?

Starting in React 18 with [`createRoot`](https://github.com/reactwg/react-18/discussions/5), all updates will be automatically batched, no matter where they originate from.

This means that updates inside of timeouts, promises, native event handlers or any other event will batch the same way as updates inside of React events. We expect this to result in less work rendering, and therefore better performance in your applications:

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 18 and later DOES batch these:
      setCount(c => c + 1);
      setFlag(f => !f);
      // React will only re-render once at the end (that's batching!)
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
-   ✅ [Demo: React 18 with `createRoot` batches even outside event handlers!](https://codesandbox.io/s/morning-sun-lgz88?file=/src/index.js) (Notice one render per click in the console!)
-   🟡 [Demo: React 18 with legacy `render` keeps the old behavior](https://codesandbox.io/s/jolly-benz-hb1zx?file=/src/index.js) (Notice two renders per click in the console.)

> Note: It is expected that you will [upgrade to `createRoot`](https://github.com/reactwg/react-18/discussions/5) as part of adopting React 18. The old behavior with `render` only exists to make it easier to do production experiments with both versions.

React will batch updates automatically, no matter where the updates happen, so this:

```jsx
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}
```
behaves the same as this:

```jsx
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

behaves the same as this:

```jsx
fetch(/*...*/).then(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
})
```

behaves the same as this:

```jsx
elm.addEventListener('click', () => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
});
```

> Note: React only batches updates when it’s generally safe to do. For example, React ensures that **for each user-initiated event like a click or a keypress, the DOM is fully updated before the next event**. This ensures, for example, that a form that disables on submit can’t be submitted twice.

## What if I don’t want to batch?

Usually, batching is safe, but some code may depend on reading something from the DOM immediately after a state change. For those use cases, you can use `ReactDOM.flushSync()` to opt out of batching:

import { flushSync } from 'react-dom'; // Note: react-dom, not react

```jsx
import { flushSync } from 'react-dom'; // Note: react-dom, not react

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```

We don't expect this to be common.

## Does this break anything for Hooks?

If you’re using Hooks, we expect automatic batching to "just work" in the vast majority of cases. (Tell us if it doesn't!)

## Does this break anything for Classes?

Keep in mind that updates _during_ React event handlers have always been batched, so for those updates there are no changes.

There is an edge cases in class components where this can be an issue.

Class components had an implementation quirk where it was possible to synchronously read state updates inside of events. This means you would be able to read `this.state` between the calls to `setState`:

```jsx
handleClick = () => {
  setTimeout(() => {
    this.setState(({ count }) => ({ count: count + 1 }));

    // { count: 1, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

In React 18, this is no longer the case. Since all of the updates even in `setTimeout` are batched, React doesn’t render the result of the first `setState` synchronously—the render occurs during the next browser tick. So the render hasn’t happened yet:

```jsx
handleClick = () => {
  setTimeout(() => {
    this.setState(({ count }) => ({ count: count + 1 }));

    // { count: 0, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

See [sandbox](https://codesandbox.io/s/interesting-rain-hkjqw?file=/src/App.js).

If this is a blocker to upgrading to React 18, you can use `ReactDOM.flushSync` to force an update, but we recommend using this sparingly:

```jsx
handleClick = () => {
  setTimeout(() => {
    ReactDOM.flushSync(() => {
      this.setState(({ count }) => ({ count: count + 1 }));
    });

    // { count: 1, flag: false }
    console.log(this.state);

    this.setState(({ flag }) => ({ flag: !flag }));
  });
};
```

See [sandbox](https://codesandbox.io/s/hopeful-minsky-99m7u?file=/src/App.js).

This issue doesn't affect function components with Hooks because setting state doesn't update the existing variable from `useState`:

```jsx
function handleClick() {
  setTimeout(() => {
    console.log(count); // 0
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
    console.log(count); // 0
  }, 1000)
```

While this behavior may have been surprising when you adopted Hooks, it paved the way for automated batching.

## What about `unstable_batchedUpdates`?

Some React libraries use this undocumented API to force `setState` outside of event handlers to be batched:

```jsx
import { unstable_batchedUpdates } from 'react-dom';

unstable_batchedUpdates(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
});
```

This API still exists in 18, but it isn't necessary anymore because batching happens automatically. We are not removing it in 18, although it might get removed in a future major version after popular libraries no longer depend on its existence.

----
----

# New API: startTransition 

> [Demo](https://react-fractals-git-react-18-swizec.vercel.app/)
> [DeepDive](https://swizec.com/blog/a-better-react-18-starttransition-demo/)
> [RealWorld](https://github.com/reactwg/react-18/discussions/65)

In React 18 we’re introducing a new API that helps keep your app responsive even during large screen updates. This new API lets you substantially improve user interactions by marking specific updates as “transitions”. React will let you provide visual feedback during a state transition and keep the browser responsive while a transition is happening.


## What problem does this solve?

Building apps that feel fluid and responsive is not always easy. Sometimes, small actions like clicking a button or typing into an input can cause a lot to happen on screen. This can cause the page to freeze or hang while all of the work is being done.

For example, consider typing in an input field that filters a list of data. You need to store the value of the field in state so that you can filter the data and control the value of that input field. Your code may look something like this:

```jsx
// Update the input value and search results
setSearchQuery(input);
```

Here, whenever the user types a character, we update the input value and use the new value to search the list and show the results. For large screen updates, this can cause lag on the page while everything renders, making typing or other interactions feel slow and unresponsive. Even if the list is not too long, the list items themselves may be complex and different on every keystroke, and there may be no clear way to optimize their rendering.

Conceptually, the issue is that there are two _different_ updates that need to happen. The first update is an urgent update, to change the value of the input field and, potentially, some UI around it. The second, is a less urgent update to show the results of the search.

```jsx
// Urgent: Show what was typed
setInputValue(input);

// Not urgent: Show the results
setSearchQuery(input);
```

Users expect the first update to be immediate because the native browser handling for these interactions is fast. But the second update may be a bit delayed. Users don't expect it to complete immediately, which is good because there may be a lot of work to do. (In fact, developers often artificially _delay_ such updates with techniques like debouncing.)

Until React 18, all updates were rendered urgently. This means that the two state states above would still be rendered at the same time, and would still block the user from seeing feedback from their interaction until everything rendered. What we’re missing is a way to tell React which updates are urgent, and which are not.

## How does startTransition help?

The new `startTransition` API solves this issue by giving you the ability to mark updates as “transitions”:

```jsx
import { startTransition } from 'react';


// Urgent: Show what was typed
setInputValue(input);

// Mark any state updates inside as transitions
startTransition(() => {
  // Transition: Show the results
  setSearchQuery(input);
});
```

Updates wrapped in `startTransition` are handled as non-urgent and will be interrupted if more urgent updates like clicks or key presses come in. If a transition gets interrupted by the user (for example, by typing multiple characters in a row), React will throw out the stale rendering work that wasn’t finished and render only the latest update.

Transitions lets you keep most interactions snappy even if they lead to significant UI changes. They also let you avoid wasting time rendering content that's no longer relevant.

## What is a transition?

We classify state updates in two categories:

-   **Urgent updates** reflect direct interaction, like typing, clicking, pressing, and so on.
-   **Transition updates** transition the UI from one view to another.

Urgent updates like typing, clicking, or pressing, need immediate response to match our intuitions about how physical objects behave. Otherwise they feel “wrong”. However, transitions are different because the user doesn’t expect to see every intermediate value on screen.

For example, when you select a filter in a dropdown, you expect the filter button itself to respond immediately when you click. However, the actual results may transition separately. A small delay would be imperceptible and often expected. And if you change the filter again before the results are done rendering, you only care to see the latest results.

In a typical React app, most updates are conceptually transition updates. But for backwards compatibility reasons, transitions are opt-in. By default, React 18 still handles updates as urgent, and you can mark an update as a transition by wrapping it into `startTransition`.

## How is it different from setTimeout?

A common solution to the above problem is to wrap the second update in setTimeout:

```jsx
// Show what you typed
setInputValue(input);

// Show the results
setTimeout(() => {
  setSearchQuery(input);
}, 0);
```

This will delay the second update until after the first update is rendered. Throttling and debouncing are common variations of this technique.

One important difference is that `startTransition` is not scheduled for later like `setTimeout` is. It executes immediately. The function passed to `startTransition` runs synchronously, but any updates inside of it are marked as “transitions”. React will use this information later when processing the updates to decide how to render the update. This means that we start rendering the update earlier than if it were wrapped in a timeout. On a fast device, there would be very little delay between the two updates. On a slow device, the delay would be larger but the UI would remain responsive.

Another important difference is that a large screen update inside a `setTimeout` will still lock up the page, just later after the timeout. If the user is still typing or interacting with the page when the timeout fires, they will still be blocked from interacting with the page. But state updates marked with `startTransition` are interruptible, so they won’t lock up the page. They lets the browser handle events in small gaps between rendering different components. If the user input changes, React won’t have to keep rendering something that the user is no longer interested in.

Finally, because `setTimeout` simply delays the update, showing a loading indicator requires writing asynchronous code, which is often brittle. With transitions, React can track the pending state for you, updating it based on the current state of the transition and giving you the ability to show the user loading feedback while they wait.

## What do I do while the transition is pending?

As a best practice, you’ll want to inform the user that there is work happening in the background. For that, we provide a Hook with an `isPending` flag for transitions:

```jsx
import { useTransition } from 'react';


const [isPending, startTransition] = useTransition();
```
The `isPending` value is true while the transition is pending, allowing you to show an inline spinner while the user waits:

```jsx
{isPending && <Spinner />}
```

The state update that you wrap in a transition doesn't have to originate in the same component. For example, a spinner inside the search input can reflect the progress of re-rendering the search results.

## Why not just write faster code?

Writing faster code and avoiding unnecessary re-renders are still good ways to optimize performance. Transitions are complementary to that. They let you keep the UI responsive even during significant visual changes — for example, when showing a new screen. That's hard to optimize with existing strategies. Even when there's no unnecessary re-rendering, transitions still provide a better user experience than treating every single update as urgent.

## Where can I use it?

You can use `startTransition` to wrap any update that you want to move to the background. Typically, these type of updates fall into two categories:

-   Slow rendering: These updates take time because React needs to perform a lot of work in order to transition the UI to show the results. [Here is a real-world demo of adding `startTransition` to keep the app responsive in the middle of an expensive re-render.](https://github.com/reactwg/react-18/discussions/65)
-   Slow network: These updates take time because React is waiting for some data from the network. This use case is tightly integrated with Suspense.

We’ll be posting again soon to cover each of these use cases with specific examples. Let us know if you have any questions!

----
----


# New Feature: SSR + Suspense

> [SOURCE](https://github.com/reactwg/react-18/discussions/37)

React 18 will include architectural improvements to React server-side rendering (SSR) performance. These improvements are substantial and are the culmination of several years of work. Most of these improvements are behind-the-scenes, but there are some opt-in mechanisms you’ll want to be aware of, especially if you _don’t_ use a framework.

The primary new API is `pipeToNodeWritable`, which you can read about in [Upgrading to React 18 on the Server](https://github.com/reactwg/react-18/discussions/22). We plan to write more about it in detail as it's not final and there are things to work out.

The primary existing API is `<Suspense>`.

**This page is a high-level overview of the new architecture, its design, and the problems it solves.**

## tl;dr

Server-side rendering (abbreviated to “SSR” in this post) lets you generate HTML from React components on the server, and send that HTML to your users. SSR lets your users see the page’s content before your JavaScript bundle loads and runs.

SSR in React always happens in several steps:

-   On the server, fetch data for the entire app.
-   Then, on the server, render the entire app to HTML and send it in the response.
-   Then, on the client, load the JavaScript code for the entire app.
-   Then, on the client, connect the JavaScript logic to the server-generated HTML for the entire app (this is “hydration”).

The key part is that **each step had to finish for the entire app at once before the next step could start.** This is not efficient if some parts of your app are slower than others, as is the case in pretty much every non-trivial app.

React 18 lets you use `<Suspense>` to **break down your app into smaller independent units** which will go through these steps independently from each other and won’t block the rest of the app. As a result, your app’s users will see the content sooner and be able to start interacting with it much faster. The slowest part of your app won’t drag down the parts that are fast. These improvements are automatic, and you don’t need to write any special coordination code for them to work.

This also means that `React.lazy` "just works" with SSR now. Here's a [demo](https://codesandbox.io/s/festive-star-9hfqt?file=/src/App.js).

_(If you don’t use a framework, you will need to change the exact way the HTML generation is [wired up](https://codesandbox.io/s/festive-star-9hfqt?file=/server/render.js:1043-1575).)_

## What Is SSR?

When the user loads your app, you want to show a fully interactive page as soon as possible:

[![](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)

This illustration uses the green color to convey that these parts of the page are interactive. In other words, all their JavaScript event handlers are already attached, clicking buttons can update the state, and so on.

However, the page can’t be interactive before the JavaScript code for it fully loads. This includes both React itself and your application code. For non-trivial apps, much of the loading time will be spent downloading your application code.

If you don’t use SSR, the only thing the user will see while JavaScript is loading is a blank page:

[![](https://camo.githubusercontent.com/7fac45f105cd741a94db77234465c4c85843b1e6f902b21bbdb1fe5b52d25a05/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f39656b30786570614f5a653842764679503244652d773f613d6131796c464577695264317a79476353464a4451676856726161375839334c6c726134303732794c49724d61)](https://camo.githubusercontent.com/7fac45f105cd741a94db77234465c4c85843b1e6f902b21bbdb1fe5b52d25a05/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f39656b30786570614f5a653842764679503244652d773f613d6131796c464577695264317a79476353464a4451676856726161375839334c6c726134303732794c49724d61)

This is not great, and this is why we recommend using SSR. SSR lets you render your React components _on the server_ into HTML and send it to the user. HTML is not very interactive (aside from simple built-in web interactions like links and form inputs). However, it lets the user _see something_ while the JavaScript is still loading:

[![](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)

Here, the grey color illustrates that these parts of the screen are not fully interactive yet. The JavaScript code for your app has not loaded yet, so clicking buttons doesn’t do anything. But especially for content-heavy websites, SSR is extremely useful because it lets users with worse connections start reading or looking at the content while JavaScript is loading.

When both React and your application code loads, you want to make this HTML interactive. You tell React: “Here’s the `App` component that generated this HTML on the server. Attach event handlers to that HTML!” React will render your component tree in memory, but instead of generating DOM nodes for it, it will attach all the logic to the existing HTML.

**This process of rendering your components and attaching event handlers is known as “hydration”.** It’s like watering the “dry” HTML with the “water” of interactivity and event handlers. (Or at least, that’s how I explain this term to myself.)

After hydration, it’s “React as usual”: your components can set state, respond to clicks, and so on:

[![](https://camo.githubusercontent.com/98a383f6de8ee2bde7516dc540aae6d9bb02a074c43c201ef6746bf3b8450420/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f715377594e765a58514856423970704e7045344659673f613d6c3150774c4844306b664a61495971474930646a53567173574a345544324c516134764f6a6f4b7249785161)](https://camo.githubusercontent.com/98a383f6de8ee2bde7516dc540aae6d9bb02a074c43c201ef6746bf3b8450420/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f715377594e765a58514856423970704e7045344659673f613d6c3150774c4844306b664a61495971474930646a53567173574a345544324c516134764f6a6f4b7249785161)

You can see that SSR is kind of a “magic trick”. It doesn’t make your app fully interactive faster. Rather, it lets you show a non-interactive version of your app sooner, so that the user can look at the static content while they wait for JS to load. However, this trick makes a huge difference for people with poor network connections, and improves the perceived performance overall. It also helps you with search engine ranking, both due to easier indexing and due to better speed.

_Note: don’t confuse SSR with Server Components. Server Components are a more experimental feature that is still in research and likely won’t be a part of the initial React 18 release. You can learn about Server Components [here](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html). Server Components are complementary to SSR, and will be a part of the recommended data fetching approach, but this post is not about them._

## What Are the Problems with SSR Today?

The approach above works, but in many ways it’s not optimal.

### You have to fetch everything before you can show anything

One problem with SSR today is that it does not allow components to “wait for data”. With the current API, by the time you render to HTML, you must already have all the data ready for your components on the server. This means that **you have to collect _all_ the data on the server before you can start sending _any_ HTML to the client.** This is quite inefficient.

For example, let’s say you want to render a post with comments. The comments are important to show early, so you want to include them in the server HTML output. But your database or API layer is slow, which is out of your control. Now you have to make some hard choices. If you exclude them from the server output, the user won’t see them until JS loads. But if you include them in the server output, you have to delay sending the rest of the HTML (for example, the navigation bar, the sidebar, and even the post content) until the comments have loaded and you can render the full tree. This is not great.

> As a side note, some data fetching solutions repeatedly try to render the tree to HTML and throw away the result until the data has been resolved because React doesn't provide a more ergonomic option. We’d like to provide a solution that doesn’t require such extreme compromises.

### You have to load everything before you can hydrate anything

After your JavaScript code loads, you’ll tell React to “hydrate” the HTML and make it interactive. React will “walk” the server-generated HTML while rendering your components, and attach the event handlers to that HTML. For this to work, the tree produced by your components in the browser must match the tree produced by the server. Otherwise React can’t “match them up!” A very unfortunate consequence of this is that **you have to load the JavaScript for _all_ components on the client before you can start hydrating _any_ of them.**

For example, let’s say that the comments widget contains a lot of complex interaction logic, and it takes a while to load JavaScript for it. Now you have to make a hard choice again. It would be good to render comments on the server to HTML in order to show them to the user early. But because hydration can only be done in a single pass today, you can’t start hydrating the navigation bar, the sidebar, and the post content until you’ve loaded the code for the comments widget! Of course, you could use code splitting and load it separately, but you would have to exclude comments from the server HTML. Otherwise React won’t know what to do with this chunk of HTML (where’s the code for it?) and delete it during hydration.

### You have to hydrate everything before you can interact with anything

There is a similar issue with hydration itself. Today, React hydrates the tree in a single pass. This means that once it’s started hydrating (which is essentially calling your component functions), React won’t stop until it’s finished doing this for the entire tree. As a consequence, **you have to wait for _all_ components to be hydrated before you can interact with _any_ of them.**

For example, let’s say the comments widget has expensive rendering logic. It might work fast on your computer, but on a low-end device running all of that logic is not cheap and may even lock up the screen for several seconds. Of course, ideally we wouldn’t have such logic on the client at all (and that’s something that Server Components can help with). But for some logic it’s unavoidable because it determines what the attached event handlers should do and is essential to interactivity. As a result, once the hydration starts, the user can’t interact with the navigation bar, the sidebar, or the post content, until the full tree is hydrated. For navigation, this is especially unfortunate since the user may want to navigate away from this page altogether—but since we’re busy hydrating, we’re keeping them on the current page they no longer care about.

### How can we solve these problems?

There is one thing in common between these problems. They force you to choose between doing something early (but then hurting UX because it blocks all other work), or doing something late (but hurting UX because you’ve wasted time).

This is because there is a “waterfall”: fetch data (server) → render to HTML (server) → load code (client) → hydrate (client). Neither of the stages can start until the previous stage has finished for the app. This is why it’s inefficient. Our solution is to break the work apart so that **we can do each of these stages for a part of the screen instead of entire app.**

This is not a novel idea: for example, [Marko](https://tech.ebayinc.com/engineering/async-fragments-rediscovering-progressive-html-rendering-with-marko/) is one of JavaScript web frameworks that implements a version of this pattern. The challenge was in how to adapt a pattern like this to the React programming model. It took a while to solve. We introduced the `<Suspense>` component for this purpose in 2018. When we introduced it, we only supported it for lazy-loading code on the client. But the goal was to integrate it with the server rendering and solve these problems.

Let’s see how to use `<Suspense>` in React 18 to solve these issues.

## React 18: Streaming HTML and Selective Hydration

There are two major SSR features in React 18 unlocked by Suspense:

-   **Streaming HTML** on the server. To opt into it, you’ll need to switch from `renderToString` to the new `pipeToNodeWritable` method, as [described here](https://github.com/reactwg/react-18/discussions/22).
-   **Selective Hydration** on the client. To opt into it, you’ll need to [switch to `createRoot`](https://github.com/reactwg/react-18/discussions/5) on the client and then start wrapping parts of your app with `<Suspense>`.

To see what these features do and how they solve the above problems, let’s return to our example.

### Streaming HTML before all the data is fetched

With today’s SSR, rendering HTML and hydration are “all or nothing”. First you render all HTML:

```jsx
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
   </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section>
    <!-- Comments -->
    <p>First comment</p>
    <p>Second comment</p>
  </section>
</main>

```

The client eventually receives it:

[![](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)

Then you load all the code and hydrate the entire app:

[![](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)

But React 18 gives you a new possibility. You can wrap a part of the page with `<Suspense>`.

For example, let’s wrap the comment block and tell React that until it’s ready, React should display the `<Spinner />` component:

```jsx
<Layout>
  <NavBar />
  <Sidebar />
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>

```

By wrapping `<Comments>` into `<Suspense>`, we tell React that **it doesn’t need to wait for comments to start streaming the HTML for the rest of the page.** Instead, React will send the placeholder (a spinner) instead of the comments:

[![](https://camo.githubusercontent.com/484be91b06f3f998b3bda9ba3efbdb514394ab70484a8db2cf5774e32f85a2b8/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)](https://camo.githubusercontent.com/484be91b06f3f998b3bda9ba3efbdb514394ab70484a8db2cf5774e32f85a2b8/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)

Comments are nowhere to be found in the initial HTML now:

```jsx
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
   </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section id="comments-spinner">
    <!-- Spinner -->
    <img width=400 src="spinner.gif" alt="Loading..." />
  </section>
</main>

```
The story doesn’t end here. When the data for the comments is ready on the server, React **will send additional HTML into the same stream, as well as a minimal inline `<script>` tag to put that HTML in the “right place”:**

```jsx
<div hidden id="comments">
  <!-- Comments -->
  <p>First comment</p>
  <p>Second comment</p>
</div>
<script>
  // This implementation is slightly simplified
  document.getElementById('sections-spinner').replaceChildren(
    document.getElementById('comments')
  );
</script>
```

As a result, **even before React itself loads on the client,** the belated HTML for comments will “pop in”:

[![](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)

This solves our first problem. Now you don’t have to fetch all the data before you can show anything. If some part of the screen delays the initial HTML, you don’t have to choose between delaying _all_ HTML or excluding it from HTML. You can just allow that part to “pop in” later in the HTML stream.

**Unlike traditional HTML streaming, it doesn’t have to happen in the top-down order.** For example, if the _sidebar_ needs some data, you can wrap it in Suspense, and React will emit a placeholder and continue with rendering the post. Then, when the sidebar HTML is ready, React will stream it along with the `<script>` tag that inserts it in the right place— even though the HTML for the post (which is further in the tree) has already been sent! There is no requirement that data loads in any particular order. You specify where the spinners should appear, and React figures out the rest.

_Note: for this to work, your data fetching solution needs to integrate with Suspense. Server Components will integrate with Suspense out of the box, but we will also provide a way for standalone React data fetching libraries to integrate with it._

### Hydrating the page before all the code has loaded

We can send the initial HTML earlier, but we still have a problem. Until the JavaScript code for the comments widget loads, we can’t start hydrating our app on the client. If the code size is large, this can take a while.

To avoid large bundles, you would usually use "code splitting": you would specify that a piece of code doesn't need to load synchronously, and your bundler will split it off into a separate `<script>` tag.

You can use code splitting with `React.lazy` to split off the comments code from the main bundle:

```jsx
import { lazy } from 'react';

const Comments = lazy(() => import('./Comments.js'));

// ...

<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>

```

Previously, this did not work with server rendering. (To the best of our knowledge, even popular workarounds forced you to choose between either opting out of SSR for code-split components or hydrating them after all their code loads, somewhat defeating the purpose of code splitting.)

But in React 18, `<Suspense>` lets you **hydrate the app** **before the comment widget has loaded.**

From the user’s perspective, initially they see non-interactive content that streams in as HTML:

[![](https://camo.githubusercontent.com/484be91b06f3f998b3bda9ba3efbdb514394ab70484a8db2cf5774e32f85a2b8/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)](https://camo.githubusercontent.com/484be91b06f3f998b3bda9ba3efbdb514394ab70484a8db2cf5774e32f85a2b8/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)

[![](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)](https://camo.githubusercontent.com/e44ee4be56e56e74da3b9f7f5519ca6197b24e9c34488df933140950f1b31c38/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)

Then you tell React to hydrate. **The code for comments isn’t there yet, but it’s okay:**

[![](https://camo.githubusercontent.com/4892961ac26f8b8dacbd53189a8d3fd1b076aa16fe451f8e2723528f51b80f66/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f304e6c6c3853617732454247793038657149635f59413f613d6a396751444e57613061306c725061516467356f5a56775077774a357a416f39684c31733349523131636f61)](https://camo.githubusercontent.com/4892961ac26f8b8dacbd53189a8d3fd1b076aa16fe451f8e2723528f51b80f66/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f304e6c6c3853617732454247793038657149635f59413f613d6a396751444e57613061306c725061516467356f5a56775077774a357a416f39684c31733349523131636f61)

This is an example of **Selective Hydration.** By wrapping `Comments` in `<Suspense>`, you told React that they shouldn’t block the rest of the page from streaming—and, as it turns out, from hydrating, too! This means the second problem is solved: you no longer have to wait for all the code to load in order to start hydrating. React can hydrate parts as they’re being loaded.

React will start hydrating the comments section after the code for it has finished loading:

[![](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)

Thanks to Selective Hydration, a heavy piece of JS doesn’t prevent the rest of the page from becoming interactive.

### Hydrating the page before all the HTML has been streamed

React handles all of this automatically, so you don’t need to worry about things happening in an unexpected order. For example, maybe the HTML takes a while to load even as it’s being streamed:

[![](https://camo.githubusercontent.com/484be91b06f3f998b3bda9ba3efbdb514394ab70484a8db2cf5774e32f85a2b8/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)](https://camo.githubusercontent.com/484be91b06f3f998b3bda9ba3efbdb514394ab70484a8db2cf5774e32f85a2b8/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)

If the JavaScript code loads earlier than all HTML, React doesn’t have a reason to wait! It will hydrate the rest of the page:

[![](https://camo.githubusercontent.com/ee5fecf223cbbcd6ca8c80beb99dbea40ccbacf1b281f4cf8ac6970c554eefa3/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f384c787970797a66786a4f4a753475344e44787570413f613d507a6a534e50564c61394a574a467a5377355776796e56354d715249616e6c614a4d77757633497373666761)](https://camo.githubusercontent.com/ee5fecf223cbbcd6ca8c80beb99dbea40ccbacf1b281f4cf8ac6970c554eefa3/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f384c787970797a66786a4f4a753475344e44787570413f613d507a6a534e50564c61394a574a467a5377355776796e56354d715249616e6c614a4d77757633497373666761)

When the HTML for the comments loads, it will appear as non-interactive because JS is not there yet:

[![](https://camo.githubusercontent.com/4892961ac26f8b8dacbd53189a8d3fd1b076aa16fe451f8e2723528f51b80f66/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f304e6c6c3853617732454247793038657149635f59413f613d6a396751444e57613061306c725061516467356f5a56775077774a357a416f39684c31733349523131636f61)](https://camo.githubusercontent.com/4892961ac26f8b8dacbd53189a8d3fd1b076aa16fe451f8e2723528f51b80f66/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f304e6c6c3853617732454247793038657149635f59413f613d6a396751444e57613061306c725061516467356f5a56775077774a357a416f39684c31733349523131636f61)

Finally, when the JavaScript code for the comments widget loads, the page will become fully interactive:

[![](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)](https://camo.githubusercontent.com/8b2ae54c1de6c1b24d9080d2a50a68141f7f57252803543c30cc69cdd4b82fa1/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f784d50644159634b76496c7a59615f3351586a5561413f613d354748716b387a7939566d523255565a315a38746454627373304a7553335951327758516f3939666b586361)

### Interacting with the page before all the components have hydrated

There is one more improvement that happened behind the scenes when we wrapped comments in a `<Suspense>`. Now their hydration no longer blocks the browser from doing other work.

For example, let’s say the user clicks the sidebar while the comments are being hydrated:

[![](https://camo.githubusercontent.com/6cc4eeef439feb3c17d0ac09c701c0deffe170c60a039afa8c0b85d7d4b9c9ef/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f5358524b357573725862717143534a3258396a4769673f613d77504c72596361505246624765344f4e305874504b356b4c566839384747434d774d724e5036374163786b61)](https://camo.githubusercontent.com/6cc4eeef439feb3c17d0ac09c701c0deffe170c60a039afa8c0b85d7d4b9c9ef/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f5358524b357573725862717143534a3258396a4769673f613d77504c72596361505246624765344f4e305874504b356b4c566839384747434d774d724e5036374163786b61)

In React 18, hydrating content inside Suspense boundaries happens with tiny gaps in which the browser can handle events. Thanks to this, **the click is handled immediately, and the browser doesn’t appear stuck** during a long hydration on a low-end device. For example, this lets the user navigate away from the page they’re no longer interested in.

In our example, only comments are wrapped in Suspense, so hydrating the rest of the page happens in a single pass. However, we could fix this by using Suspense in more places! For example, let’s wrap the sidebar as well:

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<Spinner />}>
    <Sidebar />
  </Suspense>
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

Now _both_ of them can be streamed from the server after the initial HTML containing the navbar and the post. But this also has a consequence on hydration. Let’s say the HTML for both of them has loaded, but the code for them has not loaded yet:

[![](https://camo.githubusercontent.com/9eab3bed0a55170fde2aa2f8ac197bc06bbe157b6ee9446c7e0749409b8ed978/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f78744c50785f754a55596c6c6746474f616e504763413f613d4e617972396c63744f6b4b46565753344e374e6d625335776a39524473344f63714f674b7336765a43737361)](https://camo.githubusercontent.com/9eab3bed0a55170fde2aa2f8ac197bc06bbe157b6ee9446c7e0749409b8ed978/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f78744c50785f754a55596c6c6746474f616e504763413f613d4e617972396c63744f6b4b46565753344e374e6d625335776a39524473344f63714f674b7336765a43737361)

Then, the bundle containing the code for both the sidebar and the comments loads. React will attempt to hydrate both of them, starting with the Suspense boundary that it finds earlier in the tree (in this example, it’s the sidebar):

[![](https://camo.githubusercontent.com/6542ff54670ab46abfeb816c60c870ad6194ab15c09977f727110e270517b243/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f424333455a4b72445f72334b7a4e47684b33637a4c773f613d4778644b5450686a6a7037744b6838326f6533747974554b51634c616949317674526e385745713661447361)](https://camo.githubusercontent.com/6542ff54670ab46abfeb816c60c870ad6194ab15c09977f727110e270517b243/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f424333455a4b72445f72334b7a4e47684b33637a4c773f613d4778644b5450686a6a7037744b6838326f6533747974554b51634c616949317674526e385745713661447361)

But let’s say the user starts interacting with the comments widget, for which the code is also loaded:

[![](https://camo.githubusercontent.com/af5a0db884da33ba385cf5f2a2b7ed167c4eaf7b1e28f61dac533a621c31414b/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f443932634358744a61514f4157536f4e2d42523074413f613d3069613648595470325a6e4d6a6b774f75615533725248596f57754e3659534c4b7a49504454384d714d4561)](https://camo.githubusercontent.com/af5a0db884da33ba385cf5f2a2b7ed167c4eaf7b1e28f61dac533a621c31414b/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f443932634358744a61514f4157536f4e2d42523074413f613d3069613648595470325a6e4d6a6b774f75615533725248596f57754e3659534c4b7a49504454384d714d4561)

React will _record_ that the click happened, and prioritize hydrating the comments instead because it’s more urgent:

[![](https://camo.githubusercontent.com/f76a33458a3e698125063884035e7f126104bc2c27c30c02fe8e9ebdf3048c7b/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f5a647263796a4c49446a4a304261385a53524d546a513f613d67397875616d6c427756714d77465a3567715a564549497833524c6e7161485963464b55664f554a4d707761)](https://camo.githubusercontent.com/f76a33458a3e698125063884035e7f126104bc2c27c30c02fe8e9ebdf3048c7b/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f5a647263796a4c49446a4a304261385a53524d546a513f613d67397875616d6c427756714d77465a3567715a564549497833524c6e7161485963464b55664f554a4d707761)

After the comments have hydrated, React “replay” the recorded click event (by dispatching it again) and let your component respond to the interaction. Then, now that React has nothing urgent to do, React will hydrate the sidebar:

[![](https://camo.githubusercontent.com/64ea29524fa1ea2248ee0e721d1816387127507fd3d73a013f89266162b20fba/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f525a636a704d72424c6f7a694635625a792d396c6b773f613d4d5455563334356842386e5a6e6a4a4c3875675351476c7a4542745052373963525a354449483471644b4d61)](https://camo.githubusercontent.com/64ea29524fa1ea2248ee0e721d1816387127507fd3d73a013f89266162b20fba/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f525a636a704d72424c6f7a694635625a792d396c6b773f613d4d5455563334356842386e5a6e6a4a4c3875675351476c7a4542745052373963525a354449483471644b4d61)

This solves our third problem. Thanks to Selective Hydration, we don’t have to “hydrate everything in order to interact with anything”. **React starts hydrating everything as early as possible, and it prioritizes the most urgent part of the screen based on the user interaction.** The benefits of Selective Hydration become more obvious if you consider that as you adopt Suspense throughout your app, the boundaries will become more granular:

[![](https://camo.githubusercontent.com/dbbedbfe934b41a8b4e4ed663d66e94c3e748170df599c20e259680037bc506c/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f6c5559557157304a38525634354a39505364315f4a513f613d39535352654f4a733057513275614468356f6932376e61324265574d447a775261393739576e566e52684561)](https://camo.githubusercontent.com/dbbedbfe934b41a8b4e4ed663d66e94c3e748170df599c20e259680037bc506c/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f6c5559557157304a38525634354a39505364315f4a513f613d39535352654f4a733057513275614468356f6932376e61324265574d447a775261393739576e566e52684561)

In this example, the user clicks the first comment just as the hydration starts. React will prioritize hydrating the content of all parent Suspense boundaries, but will skip over any of the unrelated siblings. **This creates an illusion that hydration is instant because components on the interaction path get hydrated first.** React will hydrate the rest of the app right after.

In practice, you would likely add Suspense close to the root of your app:

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<BigSpinner />}>
    <Suspense fallback={<SidebarGlimmer />}>
      <Sidebar />
    </Suspense>
    <RightPane>
      <Post />
      <Suspense fallback={<CommentsGlimmer />}>
        <Comments />
      </Suspense>
    </RightPane>
  </Suspense>
</Layout>
```

With this example, the initial HTML could include the `<NavBar>` content, but the rest would stream in and hydrate in parts as soon the associated code is loaded, prioritizing the parts that the user has interacted with.

> _Note: You might be wondering how your app can work in this not-fully-hydrated state. There are a few subtle details in the design that make it work. For example, instead of hydrating each individual component separately, hydration happens for entire `<Suspense>` boundaries. Since `<Suspense>` is already used for content that doesn't appear right away, your code is resilient to its children not being immediately available. React always hydrates in the parent-first order, so the components always have their props set. React holds off from dispatching events until the entire parent tree from the point of the event is hydrated. Finally, if a parent updates in a way that causes the not-yet-hydrated HTML to become stale, React will hide it and replace it with the `fallback` you specified until the code has loaded. This ensures the tree appears consistent to the user. You don’t need to think about it, but that’s what makes it work._

## Demo

We've prepared a [demo you can try](https://codesandbox.io/s/festive-star-9hfqt?file=/src/App.js) to see how the new Suspense SSR Architecture works. It is artifically slowed down, so you can adjust the delays in `server/delays.js`:

-   `API_DELAY` lets you make the comments take longer to fetch on the server, showcasing how the rest of the HTML can be sent early.
-   `JS_BUNDLE_DELAY` lets you delay the `<script>` tags from loading, showcasing how the comment widget’s HTML "pops in" even before React and your application bundle have been downloaded.
-   `ABORT_DELAY` lets you see the server "giving up" and handing off rendering to the client if fetching on the server takes too long.

## In Conclusion

React 18 offers two major features for SSR:

-   **Streaming HTML** lets you start emitting HTML as early as you’d like, streaming HTML for additional content together with the `<script>` tags that put them in the right places.
-   **Selective Hydration** lets you start hydrating your app as early as possible, before the rest of the HTML and the JavaScript code are fully downloaded. It also prioritizes hydrating the parts the user is interacting with, creating an illusion of instant hydration.

These features solve three long-standing problems with SSR in React:

-   **You no longer have to wait for all the data to load on the server before sending HTML.** Instead, you start sending HTML as soon as you have enough to show a shell of the app, and stream the rest of the HTML as it’s ready.
-   **You no longer have to wait for all JavaScript to load to start hydrating.** Instead, you can use code splitting together with server rendering. The server HTML will be preserved, and React will hydrate it when the associated code loads.
-   **You no longer have to wait for all components to hydrate to start interacting with the page.** Instead, you can rely on Selective Hydration to prioritize the components the user is interacting with, and hydrate them early.

The `<Suspense>` component serves as an opt-in for all of these features. The improvements themselves are automatic inside React and we expect them to work with the majority of existing React code. This demonstrates the power of expressing loading states declaratively. It may not look like a big change from `if (isLoading)` to `<Suspense>`, but it's what unlocks all of these improvements.


## Comments

#### Question
just wanted to call out that it would be really useful (imo) to callout `onCompleteAll` in an example. having all the content be available without JS was a concern of ours (due to SEO) so having this option is really awesome.

> Another strategy if you know that some Suspense boundaries don't contain content that influence bots you can always start streaming once you only have those left.

this might be a stupid question, but is there a way to do this via existing APIs or is this something that would need to be implemented in userland?


#### Maintainer Answer
 Not a stupid question at all. The only way to implement that in user space is to wrap all Suspense boundaries yourself to keep track of them. If you miss one you might incorrectly send them early. So I was thinking of adding a built in API for this if it turns out to be a good strategy.

Another variant that can only be done in user space though is the inverse. You can add a wrapper that you expect to see and only start flushing once you see that wrapper. Then assume that anything else is streamable.

Regarding SEO, the content is all there in the HTML regardless but it's marked with the hidden attribute and potentially out of order a bit. I'm not sure how that's handled but it might just be fine or maybe we can come up with a strategy that is fine. The inserting is currently scripts (ideally that could be done with html too) but the content is just html so it seems like something that bots could deal with.


#### Maintainer Conclusion

The general principle of hydration here is that you can think of it as lazy. In that something only needs to hydrate at all if it's being interacted with.

However hydration is also used for other things like starting subscriptions to changes, auto playing video with more control, shifting an hscroll on a timer etc.

It's also worth hydrating just to have it warmed up so it response quicker.

That's why we also eagerly hydrate things at the lowest priority in the background too.

However sometimes some things are more important to hydrate early than others.

There's an API (which might change) called `ReactDOM.unstable_scheduleHydration(node)` that does this. It takes a DOM node and then it prioritizes the path to that node for hydration.

You can use this to do user space features like hydrate the content in viewport faster than other content, or hydrating that node that you know contains auto-playing video.


----
----

# Data Fetching Concerns
>[SOURCE](https://github.com/reactwg/react-18/discussions/35)

There are a few main concerns that come to mind in terms of data-management and concurrent rendering:

1. Data changes during rendering
2. Lazy-fetching and concurrent rendering
3.  Lazy-fetching and offscreen
4.  Prefetching and concurrent rendering

## 1. Data Changes During Rendering

This scenario primarily applies to libraries like Relay or Apollo that maintain a "store" of data that is accessed by — and kept consistent across — multiple components or roots. In synchronous rendering it was generally not possible for data in the store to change during the course of a single render: render functions should be "pure", so if rendering occurs synchronously no other code can have run to modify the store. Note that libraries are generally expected to maintain this invariant too - APIs meant to be called during render should also be pure-ish: for example Relay's `useFragment()` reads data from the store but does not modify the store.

However, concurrent rendering changes this: concurrent rendering does not complete synchronously so it's possible for other code to run in between rendering of components. This can cause "tearing": when different parts of the screen show inconsistent data. An example of how this can occur:

1.  The store says the user's name is "Joe"
2.  A screen starts rendering, and the root component reads the name as "Joe"
3.  In between units of rendering, something modifies the store and sets the user's name is "Joseph"
4.  Rendering resumes and renders a child component which reads the name as "Joseph"
5.  React finally commits the rendered UI atomically, showing the inconsistent names to the user (Joe/Joseph).

The issue is step 3, which modifies the store concurrently with rendering in a way that couldn't happen before, when rendering was synchronous. Again, note that this can only happen when concurrent rendering is enabled via new features.

There are a few main solutions:

1.  Prevent the store from being modified during render (delay publishes while rendering is ongoing)
2.  Prevent modifying the version of the store that the render is reading from (ie vending some sort of immutable snapshot for the render, while the main store can advance with new data)
3.  Allow modifications to occur but detect them after the fact and fix them.

### Relay's Approach

Relay currently uses strategy #3: we maintain a "version" counter in the store that increments every time data is published, and we remember the version of the store that each component read from during render. On commit (`useEffect`) we check if the store version is the same as during render. If not we re-read the data to see if the data the component used has changed - if so we re trigger a re-render. The `useMutableSource` hook works similarly and was developed after we created our implementation. One tradeoff with `useMutableSource` is that React understands the semantics and can fall back to a synchronous re-render if data changes. The upside is that this avoids tearing, but the downside is that this comes at the cost of delaying when the screen is updated.

We also have a good idea of what it would look like to support #2 in the data layer, but it would require some additional hooks in React. Basically, the data layer would need to understand how long a snapshot should be kept around, know when to reconcile different branches of the store together, etc. This is something we expect to continue exploring going forward.

### Recommended Pattern

We recommend that existing data libraries start by adopting `useMutableSource` in order to avoid tearing during concurrent rendering.

## 2. Lazy-Fetching and Concurrent Rendering

First, note that fetching data during render isn't ideal from a performance perspective. As we discussed in our [recent blog post](https://reactjs.org/blog/2019/11/06/building-great-user-experiences-with-concurrent-mode-and-suspense.html), lazy fetching delays the point at which an application can start fetching data, which then blocks further rendering. See the post for more details about alternative approaches that we're using internally.

However, in some situations it's just easier to lazily fetch data during rendering. As a library developer trying to support this pattern in React 18, though, we have to handle the possibility that a component may render multiple times before it commits - or it may _not_ commit! So _libraries should ensure that each render doesn't trigger a duplicate fetch, while also cancelling fetches that are no longer needed_. Libraries should _not_ require developers to handle these cases. More on how to achieve this below.

As an example, consider an application with two components: a parent that fetches data (the user's profile photo URL), and then a child that shows the profile photo. The parent suspends while fetching the data, the child suspends while the photo is loading. With concurrent rendering, this would occur as follows:

1.  Parent renders, calls `useProfilePhoto()` or similar. The data is not available so the hook fetches it and suspends.
2.  React renders the nearest Suspense fallback
3.  The fetch completes, so React tries re-rendering
4.  Parent renders, calls `useProfilePhoto()`. The data is available so it returns and the component renders.
5.  Child renders, tries to access the photo but it isn't loaded yet, so it suspends.
6.  React continues rendering the nearest Suspense fallback.
7.  The image loads, so React tries re-rendering
8.  The parent and child both render w/o suspending.
9.  React commits the final view.

Note that the parent component renders 3 times in this scenario, the child renders 2 times, and each commits once. But it's also possible that the user navigated to different content before this sequence completed, in which case the parent/child may have rendered some number of times without ever committing!

### Relay's Approach

In Relay we handle these cases with a custom cache implementation that de-dupes fetches over a short period. The `useLazyLoadQuery()` hook creates a cache key for the query/variables/fetchPolicy combination and then checks if a cache entry exists. If it does the hook either returns, throws, or suspends based on the state of the cache entry. If there's no entry yet it checks whether the query can be fulfilled and populates a new entry - either returning from cache, resolving from cache while fetching in the background, or fetching and suspending. Notably, this cache entry will be marked as temporary until some component that needs it commits - and unused temporary entries are purged from the cache after a short-ish interval (30s iirc) so that future renders don't accidentally use stale data.

### Recommended Pattern

Our experience using Suspense for data-fetching in Relay and elsewhere informed the creation of the new `<Cache>` API, to make it easier to handle these cases in data-fetching libraries. The challenge for libraries is that they don't know exactly how long to cache each fetch - we don't want to fetch each time a component re-renders, but we also don't want to cache an entry so long that an unrelated fetch later in a session uses stale data. But React knows the appropriate lifetime for such cache entries, and `<Cache>` effectively implements the complex logic I described from Relay's implementation with more precise control over the cache expiration.

In terms of a recommendation for library authors, `<Cache>` is relatively new and we don't have a lot of experience using it in production. We anticipate trying to use `<Cache>` with Relay soon but this is definitely less stable than `useMutableSource`. So at the moment we recommend _experimenting_ with `<Cache>` but be prepared to potentially need to make adjustments as the API changes during the RC. Also note that `<Cache>` isn't meant for storing things like individual entries in a normalized database (or individual records in a GraphQL response) - it's really about de-duplicating coarse-grained side-effects during render such as HTTP requests. See [#25](https://github.com/reactwg/react-18/discussions/25) for more details about the Cache API.

##  3. Lazy-fetching and offscreen
TODO

## 4. Prefetching and concurrent rendering
TODO