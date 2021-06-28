# 🜂 SUSPENSE

> How do you explain suspense?

[#28](https://github.com/reactwg/react-18/discussions/28) is a very helpful thread with two main analogies for Suspense

-   [@Skn0tt](https://github.com/Skn0tt)'s analogy:

> Imagine a component that needs to do some asynchronous task before it can render, e.g. fetching some data.
> 
> Before Suspense, such a component would keep an isLoading state and return some kind fallback (an empty state, skeleton, spinner, ...) based on it.
> 
> With Suspense, a component can now, during rendering, shout "HALT! I'm not ready yet. Don't continue rendering me, please - here's a pager, I'll ping you when I'm ready!1"  
> React will walk up the tree to find the nearest Suspense boundary, which is set by You and contains the fallback that'll now be rendered until the pager is ringing.

-   [@gaearon](https://github.com/gaearon)'s analogy:

> Another way I like to explain it is that it's like throw / catch but for loading states. When a component says "I'm not ready" (exceptional situation, like throw), the closest Suspense block above (like catch) shows the fallback instead. Then, when the Promise resolves, we "retry" rendering.
> 
> One key difference with traditional Promise-based programming is that Promises are not exposed to the user code. You don't deal with them directly. You don't Promise.all them or .then them or anything like this. Instead of "wait for Promise to resolve and run some user code", the paradigm is "React will retry rendering when it resolves". In some ways it's simpler but it takes some "unlearning" because the user loses some control they might be used to from traditional Promise-base programming.


# 🜂 CONCURRENCY

> How do you explain concurrency?

Sy Brand explained concurrency in a video [using a metaphor of feeding cats](https://twitter.com/TartanLlama/status/1402255664286687233?s=20) 😻


Concurrency means that tasks can overlap.

Let's use phone calls as an analogy.

No concurrency means that I can only have one phone conversation at a time. If I talk to Alice, and Bob calls me, I have to finish the call with Alice before I can talk to Bob.

[![Non-overlapping calls](https://user-images.githubusercontent.com/810438/121394782-9be1e380-c949-11eb-87b0-40cd17a1a7b0.png)](https://user-images.githubusercontent.com/810438/121394782-9be1e380-c949-11eb-87b0-40cd17a1a7b0.png)

Concurrency means that I can have more than one conversation at a time. For example, I can put Alice on hold, talk to Bob for a bit, and then switch back to talking to Alice.

[![Overlapping calls](https://user-images.githubusercontent.com/810438/121394880-b4ea9480-c949-11eb-989e-06a95edb8e76.png)](https://user-images.githubusercontent.com/810438/121394880-b4ea9480-c949-11eb-989e-06a95edb8e76.png)

Note that concurrency doesn't necessarily mean I talk to _two people at once_. It just means that at any moment, I may be in multiple calls, and I choose who to talk to. For example, based on which conversation is more urgent.

Now, to translate the analogy, in React's case "phone calls" are your `setState` calls. Previously, React could only work on one state update at a time. So all updates were "urgent": once you start re-rendering, you can't stop. But with [`startTransition`](https://github.com/reactwg/react-18/discussions/41), you can mark a non-urgent update as a transition.

[![Overlapping calls](https://user-images.githubusercontent.com/810438/121396132-f760a100-c94a-11eb-959c-b95a6647d759.png)](https://user-images.githubusercontent.com/810438/121396132-f760a100-c94a-11eb-959c-b95a6647d759.png)

You can think of "urgent" `setState` updates as similar to urgent phone calls (e.g. your friend needs your help) while transitions are like relaxed conversations that can be put on hold or even interrupted if they're no longer relevant.

# 🜂 PASSIVE EFFECTS
> How do you explain passive effects?

There are two types of effects:

-   `useEffect` \= "effects"
-   `useLayoutEffect` \= "layout effects"

Sometimes when you say "effects" it's unclear if you mean both or just the first kind. This is why the first kind (`useEffect`) is sometimes called "passive".

We avoid this terminology anywhere in the docs, but we used it between in the core team discussions and maybe some comments on GitHub. I think that's where library maintainers may have picked it up. I wouldn't expect application authors to know or care about this terminology — but if you see it, all it means is `useEffect`, nothing more.


# 🜂 BATCHING AND AUTOMATIC BATCHING

> How do you explain batching and automatic batching?

## Batching

Imagine you're making a breakfast.

You go to market to get some milk. You come back. Then you realize you need granola. You go to market to get granola. Then you come back. Then you realize you need cookies. You go to market and get cookies. Finally, you can eat your breakfast.

But surely there is a more efficient way to do it? Instead of getting each item the moment you think of it, it's a good idea to compile a shopping list first. Then you go to market, get everything you need, and make your breakfast.

That's batching.

In React's case, each individual item is a `setState` call.

```jsx
setIsFetching(false);
setError(null);
setFormStatus('success')
```

React could re-render ("go to the shop") after each of these calls. But it's not efficient. It would be better to record all the state updates first. And then, when you finish updating all pieces of state, re-render once.

The challenge is _when_ React should do it. How can it know that you're done updating the state?

For event handlers, it's easy:

```jsx
function handleClick() {
  setIsFetching(false);
  setError(null);
  setFormStatus('success')
}
```

React is the thing calling your click handler.

```jsx
// Inside React
handleClick()
```

So React will re-render right after:

```jsx
// Inside React
handleClick()
updateScreen();
```

That's how React always worked.

## Automatic batching

Coming back to this example:

```jsx
setIsFetching(false);
setError(null);
setFormStatus('success')
```

What if doesn't happen during an event? For example, maybe this code runs as a result of a fetch. React has no idea when you're going to "stop" setting states. So it needs to pick some time to update the screen.

This is similar to how maybe you're not having breakfast, but just getting random cravings throughout the day. Do you go to the market the moment you feel you're hungry? Or do you wait a bit of time (say, 30 minutes), and then go to the market with a list of whatever you've come up with so far?

Previously, React used to re-render immediately in this case. This means you'd get three screen updates for three `setState` calls. That's not very efficient.

With automated batching in React 18, it will always batch them. Practically speaking, it means React will "wait a bit" (the technical term if "until the end of the task or the microtask"). Then React will update the screen.

# 🜂 HYDRATION

> How do you explain hydration?

Dan's explanation of hydration in [#37](https://github.com/reactwg/react-18/discussions/37) (under "what is SSR?" subheader) is on point 💯 Here's an excerpt:

> The process of rendering your components and attaching event handlers is known as “hydration”. It’s like watering the “dry” HTML with the “water” of interactivity and event handlers.

> The analogy I like for "hydration" is that it loads dry, and then oil is applied to the gears to move it.


# 🜂 SSR

> How do you explain ssr?

SSR can be explained well with food:

> Suppose you go to a salad bar and get each of your ingredients to make your own salad. You put them together, layer things the way you want, and then eat it. That's client-side rendering. You're putting together the pieces (HTML, CSS, JS, yourself).

> Alternatively, you can go to the salad bar and pick up a salad that's pre-made. It's all ready for you, all you have to do is eat it. That's server-side rendering. You get render ready HTML delivered directly from the server.

Most people think of React as running on the client (i.e., in the user's browser). When components get rendered, that results in changing the HTML that's on the page. When a user visits a webpage using React, first we need to download React and the JavaScript code for each React component, and then in their browser we run that code to figure out what HTML we want to show on screen. Once that happens, the user can see the app and interact with it.

Server-side rendering (SSR) is a tool designed to speed up that process so that pages using React load faster. When using SSR, we first run the components on the server that's sending the files for the page to the user. We take the HTML outputted by those components and send _that HTML_ to the user's browser. As soon as that arrives, the user can see the full content of the page. Note that the HTML by itself is a snapshot of what the page should initially look like and it's not interactive by itself. So we still need to download React and the JS code for each React component, and then we need to tell React to "take control" of the HTML that's already in the page, which is called hydration. Once that finishes, the user can see the full content of the page _and_ they can interact with it.

SSR helps make the initial page visible sooner but it doesn't make it interactive sooner because you still need to wait to download and run all the same JS code – with SSR, we're running all the components on the server now, but we're also running all of them on the client still. (Upcoming work on [Server Components](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) – a new experiment by the React team – will help make it so you can run some components _only_ on the server which can help this, but it won't be ready for a while.)

In order to use SSR, you need to do two things: (a) make sure that each React component in your codebase only uses features that work on both server and client (there are some requirements, like not using the `window` object that's only available in browsers), and (b) change your server-side code to call React and do something with the resulting HTML. Some frameworks like Next.js come with SSR support out of the box, meaning that they have done part (b) for you.

# 🜂 APP STATE, COMPONENT STATE, UI STATE
> How do you explain state and different types of state?

Your React components transform information into UI. There are two types of information.

Some of this information never changes during a user session. For example, your website layout, the icons on your buttons, and the order of sections on the page.

Other information may change in response to the user interaction. For example, is a button hovered or not. How many times the user clicked “add”. The content of their shopping cart or a message they were typing.

Information that changes in response to user interactions is called “state”.

Any information has some lifetime. It’s attached to something. For example, regular JS variables are attached to the function calls. They only live for so long as you’re inside this function. When you exit a function, variables from that call stop “existing.” (Aside from closures, which occur when you nest functions, and which let a nested function “keep seeing” variables from the outer function call.)

In React, each time a component renders, it’s a separate function call. So if you tried to store some information like shopping cart content in a regular variable, it would simply disappear on next render. This is why React has a concept of “state variables” you define with `useState`. They are associated with a place in the tree — like, a particular text input or a shopping cart you see on the screen — rather than with a function call.

Where you declare a state variable matters. Here’s how to think about it. If you render something in two places on the screen, would you want them to be synchronised or not?

For example, rendering two comment fields doesn’t mean that you want one to update when you type in the other one. So it makes sense to place state _inside_ the comment field. Each copy is independent.

But maybe you’re rendering an already submitted comment in two places. If you make an edit to one of them, and _save_ it, it would be reasonable to expect that both of them need to reflect your edit. So the information about a list of comments needs to live somewhere _outside_ your component — maybe, in their shared parent or in some kind of a cache.

This is what people mean by different kinds of state. These aren’t technical terms. But “UI state” usually means some state specific to a concrete UI widget. Like text input’s text. Whereas “app state” usually means some information that’s shared between many components and needs to be in sync. So it’s usually placed outside of them.

# 🜂 RENDERS: RENDERS, RE-RENDERS, WASTEFUL RENDERS?
> How do you explain Renders, re-renders, and wasteful renders and their impact on performance?

When the user does something (like clicking a button), you might want to change what’s on the screen. The way you do it by setting state. In response, React does a “render”.

During a render, React calls the component whose state you set. Your component returns what should be on the screen. Then, React does the same for components below that one (because they might also show something different). This process is called rendering.

This might remind you of top-down planning. A design agency might receive an order. The art director sets a vision and hands off illustration, typography, or logo design, to people specialising in that. They, in turn, might hand off parts of that work to other colleagues. And so on. In the end there is a result composed of different people’s work.

This is similar to how what ends up on the screen is composed of different components’ render results.

Let’s say the design agency client now wants to change something. “Let’s tweak the colors.” So the top-down process starts again. The art director talks to the designer who talks to the person making the sketches, who implements the changes. Then we have a new design.

That’s what a “re-render” is. It’s when you set the state again, React calls your components to figure out what should be on the screen, and then updates the screen. A “re-render” means “rendering again”, nothing more than that.

Finally, a “wasted render” means that a part of the work was unnecessary. For example, if our client says “play with the fonts a bit” and the whole agency starts another round of iterations. But actually only the art director and the font designer needed to be involved. There was no need to involve the person working on the color palette.

In React, this can happen when you update the state of a component that contains many other components. They might not be affected by update, but React still needs to ask each one of them whether they want to display something different. So it’s a “waste” if they don’t end up changing in response.

There are a few common ways to optimize “wasted” renders:

1.  You could move state deeper in the tree if it doesn’t affect some components. Then React won’t ask them.
2.  You can pass some components “from above” as children to the component whose state is updated. Then React knows those components from “above” don’t care about this state update, and don’t need to change.
3.  You could mark some components as “memoized” which means they’ll “remember” their last props and render result. Then, before re-rendering, React will check whether their props have changed. If not, it will skip them while rendering.

_PS. I know design agencies don’t necessarily work top-down like this. It’s just a metaphor._

# 🜂 DEBOUNCING AND THROTTLING
> How do you explain debouncing and throttling?

## What is debouncing and throttling?

Debouncing and throttling are too common techniques for dealing with things that happen "too often". Imagine you're meeting a friend, and they are telling you a story, but they struggle to pause when talking. Let's say you want to accommodate them while also responding to what they have to say, when possible. (I know this might be a bit contrived, but bear with me!)

Let's say you can never speak at the same time. You have a few strategies:

### Synchronous

You could respond to each sentence the moment they finish it:

[![Synchronous responses](https://user-images.githubusercontent.com/810438/121427718-f3457b00-c96c-11eb-9515-0ab2974414c9.png)](https://user-images.githubusercontent.com/810438/121427718-f3457b00-c96c-11eb-9515-0ab2974414c9.png)

This might work okay if your responses are short. But if your responses are longer, this could make it very hard for them to finish the story. So this strategy isn't great.

### Debounced

You could wait for them to stop talking. For example, if they pause for long enough, you could start responding:

[![Debounced responses](https://user-images.githubusercontent.com/810438/121427785-05271e00-c96d-11eb-88f2-b0c80221fe06.png)](https://user-images.githubusercontent.com/810438/121427785-05271e00-c96d-11eb-88f2-b0c80221fe06.png)

This strategy works well if your friend is occasionally making pauses. However, if they're talking for a few minutes without a pause, this doesn't let you respond at all:

[![Debouncing at the end](https://user-images.githubusercontent.com/810438/121428868-348a5a80-c96e-11eb-8b7d-c0b7bb45971c.png)](https://user-images.githubusercontent.com/810438/121428868-348a5a80-c96e-11eb-8b7d-c0b7bb45971c.png)

### Throttled

You could decide to respond _at most_ once a minute. Here, you keep count of how long you haven't spoken for. Once you haven't spoken for a minute, you insert your response right after the friend's next sentence:

[![Throttled responses](https://user-images.githubusercontent.com/810438/121428472-c2b21100-c96d-11eb-962c-ea1c190815cd.png)](https://user-images.githubusercontent.com/810438/121428472-c2b21100-c96d-11eb-962c-ea1c190815cd.png)

This strategy is helpful if your friend wants you to respond as they go, but they don't ever create pauses for you to do it. But it's not great if they're creating pauses but you're still waiting out for no reason:

[![Waiting for throttling](https://user-images.githubusercontent.com/810438/121429118-77e4c900-c96e-11eb-9ac1-88f3816487e6.png)](https://user-images.githubusercontent.com/810438/121429118-77e4c900-c96e-11eb-9ac1-88f3816487e6.png)

## How does this relate to React?

Friend's "sentences" are events like button clicks or keyboard typing. Your "responses" are updating the screen.

You use debouncing or throttling when the user is doing something too fast (e.g. typing), and updating the screen in response to _each_ individual event is just too slow. So you either wait for the user to stop typing (debouncing) or you update the screen once in a while, like once a second (throttling).

## What about React 18?

The interesting thing is that [`startTransition`](https://github.com/reactwg/react-18/discussions/41) makes _both_ strategies unnecessary for many cases. Let's add one more strategy that you didn't have before React 18:

### Cooperative

You start responding _immediately_ the moment your friend finishes a sentence, without waiting. But you've agreed with your friend that they may interrupt you if they want to keep going. So in that case you'll abandon your attempt to respond. Right after the next sentence, you'll try again. And so on:

[![Cooperative responses](https://user-images.githubusercontent.com/810438/121429595-ffcad300-c96e-11eb-81dc-29900d9b4cb9.png)](https://user-images.githubusercontent.com/810438/121429595-ffcad300-c96e-11eb-81dc-29900d9b4cb9.png)

(You _also_ agree that you will interrupt _them_ if enough time has passed and you still haven't managed to respond. This prevents the case where you don't get to speak for too long.)

Note how in this strategy **you're not wasting any time and _also_ don't prevent your friend from continuing**.  
This gives you the best of all three previous approaches. (Who would have thought that being cooperative helps!)

In the coding example, this means that React starts rendering _immediately_ after the user types. There's no need to delay doing it (like debouncing does). If the user types again, React will just abandon that work and start again. If rendering was fast enough to fit in the pause between events, then you haven’t wasted any time (by waiting unnecessarily). You’re also not wasting time on finishing a render that's no longer needed.

**You can see a demo of the difference [here](https://react-beta-seven.vercel.app/)**. Try typing into the input. The more you type, the more complex graph we draw (artificially slowed down to show the point). See the different behavior between the three strategies. How do they behave when as you keep typing longer text into the input?

Which one feels smoother?

# 🜂 FLUSH SYNC
> How do you explain flush sync?

`flushSync` is the name of the method that lets control when React updates the screen ("flushes" the state updates). In particular, it lets you tell React to do it _right now_ ("synchronously"):

```jsx
flushSync(() => {
  setState(something);
});
// By this line, the screen has been updated
```
"Flush synchronously" = "Update the screen now"

Normally, you shouldn't need `flushSync`. You would only use it if you need to work around some issue where React updates the screen later than you need it to. It is very rarely used in practice.

# 🜂 DISCRETE EVENTS
> How do you explain discrete events?

"Discrete" events is not a concept we use anywhere in the docs. It's not a concept we expect React application developers to know or care about.

However, it comes up when we explain how [automatic batching](https://github.com/reactwg/react-18/discussions/21) works. What we have to clarify is that for some events, like clicks, we guarantee that the screen is updated before the next event can happen. This is important for UIs like counters (imagine the user pressing the button two times very fast) or forms (where sending a form may disable it, and if it doesn't happen before the second click, you risk sending a form twice).

We call events like these "discrete". This means that each event is **separately** intended by the user. If I click twice, it's because I _mean_ to click twice. If I type "h", "e", "l", "l", "o", it's because I _mean_ to type "hello".

This is different from events like mousemove. I may move the mouse over ten different coordinates but I don't intentionally _mean_ any of these separate coordinates. As a user I have no idea how many mousemove events I just performed. I only moved the mouse. We call such events "continuous" — as in, what matters is the _latest_ one, not how many there happened.

These concepts only matter for React internally, because it may batch updates in multiple continuous events, but it would still update the screen for each discrete event in a row. However, as a React user, you probably won't need to think about this.
