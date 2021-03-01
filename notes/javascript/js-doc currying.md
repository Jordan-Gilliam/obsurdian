# currying
To make functions more modular and easier to reuse, we can use techniques like currying, which lets us take advantage of closure to turn a function with any number of arguments into a series of single-argument functions, such that we can provide only some of the input arguments and get a "partially applied" function.

```js
const add = a => b => a + b;
const eq = a => b => a === b;

eq(5)(add(2)(3));
```

**The algorithm for currying.** The function curry curries a given binary function. It has the signature

> curry: (X × Y → R) → (X → (Y → R))

**Currying doesn’t call a function. It just transforms it.**

## why curry?
-   Currying helps you to avoid passing the same variable again and again.
-   It helps to create a higher order function. It extremely helpful in event handling. See the [blog post](https://bjouhier.wordpress.com/2011/04/04/currying-the-callback-or-the-essence-of-futures/) for more information.
-   Little pieces can be configured and reused with ease.

```js
function sum(a) {
    return (b) => {
        return (c) => {
            return a + b + c
        }
    }
}

console.log(sum(1)(2)(3)) // 6
// OR
const sum1 = sum(1);
const sum2 = sum1(2);
const result = sum2(3);
console.log(result); // 6
```

___

#### generic function
> generic function which checks if the word has n characters.

```js
let hasNChars = (n=3) => (word) => word.length === n;

let words = \['forest', 'gum', 'pencil', 'wonderful', 'grace',
    'table', 'lamp', 'biblical', 'midnight', 'or', 'perseverance', 
    'adminition', 'redemption', 'dog', 'no'\];


let res = words.some(hasNChars(2), words);
console.log(res);

let res2 = words.some(hasNChars, words);
console.log(res2);
```
___
#### specialized function
> specialized functions are derived from more generic functions.

```js
let greet = (message) => (name) => {
    return \`${message} ${name}!\`;
}

let helloGreet = greet('Hello'); 

console.log(greet('Good day')('Lucia'));
console.log(helloGreet('Peter'));
```
___
#### function composition
> function composition allows to combine any number of functions to create a new one.


```javascript
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));


const sum = a => b => a + b
const multiply = a => b => a * b

const addTransactionFee = sum(2)
const addTax = multiply(1.1)
const addMonthlyPromotion = multiply(0.8)
const log = label => value => {
  console.log(label, value)
  return value
}

const paymentAmount = compose(
  log('addTransactionFee'),
  addTransactionFee, 
  log('addTax'),
  addTax,
  log('addMonthlyPromotion'),
  addMonthlyPromotion
)(100)
// Output:
// addMonthlyPromotion 80
// addTax 88
// addTransactionFee 90
```

#js #functional-programming 