# React Render Behavior
> [SOURCE](https://github.com/reactwg/react-18/discussions/7)
> [DeepDive](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/)

## Summary
-   The "render phase" is when React is looping over the component tree and asking each component to return elements (ie, `classInstance.render()`, and `FunctionComponent(props)`)
-   After React has come up with the complete rendered element tree and diffed it, it's ready to apply the changes to the DOM. This is the end of the "render phase".
-   Those changes are applied synchronously to the DOM at the start of the "commit phase"
-   React then runs the `cDM/cDU` lifecycle methods and `useLayoutEffect` hooks synchronously in the commit phase
-   "Passive effects" (the `useEffect` hook) are run on a brief timeout after the commit phase completes


## There are two phases:

### Render phase  
is when we figure out **_what_ to show on the screen**. You can think of it as just a calculation. It should not “do” anything, only calculate. It’s pure. Code-wise, it corresponds to React calling your component function bodies (and you returning JSX). No side effects at this point. When you call `useEffect`, we “remember” that effect for later — conceptually it’s like a part of the JSX you return.
    
### Commit phase
is when we **put things in the DOM**. That’s also when we run all the effects we’ve collected during the render phase.
    

## Real World Use Case:
> TLDR; when you start seeing flame graphs that are broken up or stack traces with async boundaries, that's due to the render phase yielding!


When you use Concurrent Features, we take advantage of purity of the render phase. In particular, Concurrent Features assume that it’s safe to throw away half-finished rendering results (and restart it). Or that it’s safe to do a render for a different version of props/state than is currently on the screen. As long as rendering (aka calling your component bodies) is pure, it is a safe assumption.

One place that you may see the render and commit phases come up is when you're profiling performance. For example, consider an update caused by typing into the form field from the time-slicing demo deployed [here](https://react-beta-seven.vercel.app/):

By default, React currently renders and commits updates synchronously. What that means, is that when you profile this interaction in DevTools, you'll see one long function call where React renders the tree to collect the updates and then commits the updates to the DOM all at once:

[![Screen Shot 2021-05-28 at 12 07 48 PM](https://user-images.githubusercontent.com/2440089/120013337-9f22ba00-bfae-11eb-91b9-abc7801727ee.png)](https://user-images.githubusercontent.com/2440089/120013337-9f22ba00-bfae-11eb-91b9-abc7801727ee.png)

Here we have a render and commit phase, but the difference between them is not as clear because everything is synchronous.

When you switch this example over from "synchronous" to "concurrent", the only code we change is that we wrap the update in `startTransition`. This opts-in to concurrent rendering.

In concurrent rendering, React will yield during the render phase to other work in the browser, and to other React work at different priorities. This yielding allows other work to be done, which may schedule different updates at higher priority. For example, a promise may resolve and call setState to show new results on screen, which may be higher priority than other work that's not visible. When everything was synchronous, the promise wouldn't be handled until after the update finished, pushing out when this data is visible to the user. Yielding allows the promise to be handled while we're rendering, and tell React that we need to interrupt the current work for a higher priority update.

So when you profile this same interaction using a concurrent feature, the difference between the two phases is more clear. Now, the render phase is broken up into multiple tasks of smaller batches of work. In this phase we're collecting all of the updates we need to make and yielding every 5ms. Once we have all the updates, we start the commit phase. Here, we synchronously make all of the updates we need to make to the DOM.

This results in a flame graph that looks like this:

[![Screen Shot 2021-05-28 at 12 38 40 PM](https://user-images.githubusercontent.com/2440089/120015721-a8615600-bfb1-11eb-98b2-a2f8cf02d301.png)](https://user-images.githubusercontent.com/2440089/120015721-a8615600-bfb1-11eb-98b2-a2f8cf02d301.png)

Note that this example is dramatized to show the effect. Generally, most of the work happens in the render phase followed by a relatively short commit. The point is, when you start seeing flame graphs that are broken up or stack traces with async boundaries, that's due to the render phase yielding!