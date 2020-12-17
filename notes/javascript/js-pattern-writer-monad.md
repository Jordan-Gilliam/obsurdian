# Writer Monad
> Another common situation is keeping a log file or otherwise reporting a program's progress. 

Sometimes, a programmer may want to log even more specific, technical data for later profiling or debugging. The Writer monad can handle these tasks by generating auxiliary output that accumulates step-by-step.

To show how the monad pattern is not restricted to primarily functional languages, this example implements a Writer monad in JavaScript. 


First, an array (with nested tails) allows constructing the Writer type as a linked list. The underlying output value will live in position 0 of the array, and position 1 will implicitly hold a chain of auxiliary notes:

```js 
const writer = [value, []];
```

Defining unit is also very simple:

```js
const unit = value => [value, []];
```

Only unit is needed to define simple functions that output Writer objects with debugging notes:

```js
const squared = x => [x * x, [`${x} was squared.`]];
const halved = x => [x / 2, [`${x} was halved.`]];
```

A true monad still requires bind, but for Writer, this amounts simply to appending a function's output to the monad's linked list:

```js
const bind = (writer, transform) => {
    const [value, log] = writer;
    const [result, updates] = transform(value);
    return [result, log.concat(updates)];
};
```

The sample functions can now be chained together using bind, but defining a version of monadic composition (called pipelog here) allows applying these functions even more succinctly:

```js
const pipelog = (writer, ...transforms) => transforms.reduce(bind, writer);
```

The final result is a clean separation of concerns between stepping through computations and logging them to audit later:

```js
pipelog(unit(4), squared, halved);
// Resulting writer object = [8, ['4 was squared.', '16 was halved.']]
```

#js #pattern