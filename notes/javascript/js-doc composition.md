# composition
> function composition allows to combine any number of functions to create a new one.

construct programs entirely out of modular pure functions, using _function composition_ to "combine" several functions' effects to create a pipeline through which our program's data can flow.

## Pipelining
```javascript
function pipeline(...fns) {
  if (length(fns) === 0) return input => input;
  if (length(fns) === 1) return input => head(fns)(input);
  return function(input) {
    return pipeline(...tail(fns))(head(fns)(input));
  };
}
```

#### reduce pipeline 
```js
const pipe = (...fns) => input => reduce((acc, fn) => fn(acc), input, fns)
```

___

## use case

 let's explore a _case_ in which they're useful!

Different programming languages use different [conventions](https://en.wikipedia.org/wiki/Naming_convention_(programming)#Multiple-word_identifiers) for variable names. For example, [`snake_case`](https://en.wikipedia.org/wiki/Snake_case) variables are common in Python, whereas [`camelCase`](https://en.wikipedia.org/wiki/Camel_case) variables are common in JavaScript.

The `snakeToCamel` function makes a composed function that reformats a string from `snake_case` to `camelCase`. 

-   Break down the transformation into small single-operation functions, then use your `pipeline` function to compose them. 

```js
// Takes a "snake_case_string" and returns a split array of the words
function desnake(snake_case_string) {
  return snake_case_string.split('_');
}

// Takes a "string" and returns a string with the first letter capitalized
function capitalizeFirstLetter(string) {
  return string.charAt(0).toUpperCase() + string.substr(1).toLowerCase();
}

// Takes an ["array", "of", "strings"] and returns a camelized ["array", "Of", "Strings"]
function capitalizeAll(stringArray) {
  return map(capitalizeFirstLetter, stringArray);
}

function camelize(stringArray) {
  return [head(stringArray)].concat(capitalizeAll(tail(stringArray)));
}

function concatenate(stringArray) {
  return reduce((acc, str) => acc + str, '', stringArray);
}

function hyphenate(stringArray) {
  return reduce(
    (acc, str) => [acc, str].join('-'),
    head(stringArray),
    tail(stringArray)
  );
}

// Takes a "snake_case_string" and returns a "camelCaseString"
function snakeToCamel(snake_case_string) {
  const pipe = pipeline(desnake, camelize, concatenate);
  return pipe(snake_case_string);
}

// Takes a "snake_case_string" and returns a "Train-Case-String"
snakeToTrain = pipeline(desnake, capitalizeAll, hyphenate)
```