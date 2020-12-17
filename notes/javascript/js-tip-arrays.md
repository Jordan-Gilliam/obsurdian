# Array API

```js
const jsStuff = [{
  operator: 'switch',
  isAnyGood: false
}, {
  operator: 'var',
  isAnyGood: false
}, {
  operator: 'for',
  isAnyGood: false
}, {
  operator: 'const',
  isAnyGood: true
}, {
  operator: '...',
  isAnyGood: true
}];

// ❌ bad way, using variables and FOR Loops:
let theGoodPart = [];
for (let item of jsStuff) {
  if (true === item.isAnyGood) {
    theGoodPart.push(item);
  }
}

// ✅ good way, using Array API and Function Composition:
const isGood = (item) => (true === item.isAnyGood);
const operatorOnly = (item) => item.operator;
const theGoodPart = jsStuff
  .filter(isGood)
  .map(operatorOnly);
```
  
  #js #tip