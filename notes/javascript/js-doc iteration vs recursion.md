# iteration vs recursion
> In functional programming, we avoid mutable state, and therefore avoid iterative loops using `for` or `while`. As an alternative to iteration, we use _recursion_ to break down the problem into smaller ones.

A recursive function has two parts:

-   **Base case**: condition(s) under which the function returns an output without making a recursive call 
-   **Recursive case**: condition(s) under which the function calls itself to return the output

#### Iterative
```js
function iterativeFibonacci(n) {
  if (n === 0) return 0;
  if (n === 1) return 1;

  let previous = 0;
  let current = 1;
  for (let i = n; i > 1; i--) {
    let next = previous + current;
    previous = current;
    current = next;
  }
  return current;
}

iterativeFibonacci(2)  // 1
iterativeFibonacci(20)  // 6765
```

#### recursive
```js
function recursiveFibonacci(n) {
  if (n === 0) return 0;
  if (n === 1) return 1;
  return recursiveFibonacci(n - 2) + recursiveFibonacci(n - 1);
}

recursiveFibonacci(2)  // 1
recursiveFibonacci(20)  // 6765
```


#js #functional-programming  #pattern 