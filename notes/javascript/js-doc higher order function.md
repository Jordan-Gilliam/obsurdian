# higher order function

> higher order functions receive another function as an input or output

___

## filter
The `filter` function takes a `predicate fn`(a function that takes in a value and returns a boolean) and an `array`.

It applies the predicate function to each value in the array, and returns a `new array` with only those values for which the predicate function returns `true`. 

```js
const wholes = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

function filter(predicateFn, array) {
  if (length(array) === 0) return [];
  const firstItem = head(array);
  const filteredFirst = predicateFn(firstItem) ? [firstItem] : [];
  return concat(filteredFirst, filter(predicateFn, tail(array)));
}

function isPrime(n) {
  if (n <= 1) return false;
  const wholes = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  const possibleFactors = filter(m => m > 1 && m < n, wholes);
  const factors = filter(m => n % m === 0, possibleFactors);
  return factors.length === 0 ? true : false;
}

prime = filter(isPrime, wholes) // [2,3,5,7]
```

___

## map
The `map` function takes a one-argument function and an array, and applies the function to each element in the array, returning a new array of the resulting values. 

To move towards a functional mindset, use these helper functions instead of the equivalent object-oriented array methods:

-   `head(array)` to return the first element of an array (e.g. `head([1,2,3]) -> 1`)
-   `tail(array)` to return the rest of the array after the first element (e.g. `tail([1,2,3])` returns `[2,3]`)
-   `length(array)` to return the number of elements in the array (e.g. `length([1,2,3])` returns `3`)

```js
const wholes = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

function map(fn, array) {
  if (length(array) === 0) return [];
  return [fn(head(array))].concat(map(fn, tail(array)));
}

fizzBuzz = map(n => {
  const fizzed = n % 3 === 0 ? 'fizz' : '';
  const buzzed = n % 5 === 0 ? 'buzz' : '';
  return fizzed || buzzed ? fizzed + buzzed : n;
}, wholes) // [fizzbuzz,1,2,fizz,4,buzz,fizz,7,8,fizz,buzz]
```

___

## reduce
The `reduce` function is the odd one of the bunch. Unlike `filter` and `map`, which each take an array and return another array, `reduce` takes in an array and returns a single value - in other words, it "reduces" an array to a single value. 

`reduce` takes three arguments:

-   a "reducer" function, which takes two arguments - an accumulator and the next value from the array - and returns a single value. This function will be applied to each value in the array, with the accumulator storing the reduced value so far.
-   an initial value, passed to the first call of the reducer function
-   the array to reduce

Take a moment to read the recursive implementation of `reduce` below:

```js
const wholes = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

function reduce(reducerFn, initialValue, array) {
  if (length(array) === 0) return initialValue;
  const newInitialValue = reducerFn(initialValue, head(array));
  return reduce(reducerFn, newInitialValue, tail(array));
}

sum = reduce((acc, val) => acc + val, 0, wholes)
max = reduce((acc, val) => (val > acc ? val : acc), 0, wholes)
```