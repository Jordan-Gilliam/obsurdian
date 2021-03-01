# closure
closure occurs when a function defined inside another function retains access to scope from the outer function's body. This allows the inner function to access data that was not explicitly passed to it as an argument.

```js
function makeAdjectifier(adjective) {
	return function(noun) {
		return adjective + " " + noun;
	};
}

const dopeify = makeAdjectifier("dopee");
dopeify("deal on those jeans"); // "dopee deal on those jeans"
dopeify("album art"); // "dopee album art"
```

#js #functional-programming 