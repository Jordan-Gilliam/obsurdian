# 👉 Avoid Tedious Conditionals
> In modern web development, you can use try...catch to simplify tedious checks like:

```js
// ❌ bad way, using tedious checks:
let val = null;
if (myObject.k1 && myObject.k1.k2 && myObject.k1.k2.k3) {
  val = myObject.k1.k2.k3;
}

// ✅ good way, using try...catch:
let val = null;
try {
  val = myObject.k1.k2.k3;
} catch () {
  val = null;
}

// 😎 cool way, with const, immediate expression & arrow functions:
const val = (() => {
  try {
    return myObject.k1.k2.k3;
  } catch () {
    return null;
  }
})();
```

#js #tip