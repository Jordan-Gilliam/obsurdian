# partial application
Partial application takes a function

> f: X × Y → R

and a fixed value x for the first argument to produce a new function

> f': Y → R

**The algorithm for partial application.** The function partApply partially applies binary functions. It has the signature

> partApply : ((X × Y → R) × X) → (Y → R)

```js
const partial = (f, ...args) => (...args_) =>
  f(...args, ...args_);

const sum = (a, b, c) =>
  a * b * c;

partial(sum, 1) (2, 3); // 6

partial(sum, 1) (2, 3); // 6
```

#js #functional-programming  