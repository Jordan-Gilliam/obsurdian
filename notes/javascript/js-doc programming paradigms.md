## Imperative 
> follow my commands: do this, then that

## Declarative
> this is what I want do it however you want

## Object-Oriented
> keep state to yourself: send/receive messages

## Functional
> pure functions: only input in -> only output out 

### Not pure
```js
let name = "Jordan"

function greet() {
	console.log(`Hello, ${name}!`)
}

greet(); // Hello, Jordan!

name = "Koolaid Man"

greet() // Hello, Koolaid Man!
```

#### examples

```js
function setColor(R, G, B) {
  const hex = rgbToHex(R, G, B);
  const colorMe = document.getElementById('color-me');
  colorMe.setAttribute('style', 'color: ' + hex);
}
```
A function is not pure if it does anything besides return its output. Any other effect it has on the program or world is a side effect (in this case, changing the properties of an HTML element on the page).


```js
async function readJsonFile(filename) {
  const file = await fetch(
    'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_hour.geojson'
  );
  return await file.json();
}
```
A function is not pure if its output depends on the state of the world (in this case, the contents of web-hosted file), or if calling the function at different times with the same inputs can give different outputs.

___

### Pure
A function is pure if its output depends on nothing but its inputs, it does nothing except return its output, and it always returns the same output if called with the same input.
```js
function greet(name) {
	return `Hello, ${name}!`
}

greet("Jordan") // Hello, Jordan!
greet("Koolaid Man") // Hello, Koolaid Man!
```

#### examples
```js
function toHex(n) {
  let hex = n.toString(16);
  return hex.padStart(2, '0');
}
```

```js
function writeJsonString(object) {
  return JSON.stringify(object, null, 2);
}
```

```js
function exclusiveOr(A, B) {
  return (A || B) && !(A && B);
}
```

#pattern #architecture 