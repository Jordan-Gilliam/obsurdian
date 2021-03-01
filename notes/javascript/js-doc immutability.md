# immutability
> don't change in-plate; instead, replace

## mutation ❌
```js
let countries = ["Finland", "Canada", "Austrailia"];
countries[1] = "Sweden";
```

## no mutation ✅
```js
const oldArr = ["Finland", "Canada","Austrailia"];
const newArr = oldArr.map(country => {
	if (country === "Finland") return "Sweden";
	if (country === "Canada") return "Germany";
	return country;
})
```

___


## persistent data structures
> use "structural sharing" to reuse unchange data

```js
// instead of myArray[index] = value
function update(index, value, array) {
  return array
    .slice(0, index)
    .concat([value])
    .concat(array.slice(index + 1));
}
```

... use [immer](https://immerjs.github.io/immer/docs/introduction) for now

#js #data-structures #functional-programming 